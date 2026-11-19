This file is a merged representation of a subset of the codebase, containing specifically included files, combined into a single document by Repomix.

# File Summary

## Purpose
This file contains a packed representation of a subset of the repository's contents that is considered the most important context.
It is designed to be easily consumable by AI systems for analysis, code review,
or other automated processes.

## File Format
The content is organized as follows:
1. This summary section
2. Repository information
3. Directory structure
4. Repository files (if enabled)
5. Multiple file entries, each consisting of:
  a. A header with the file path (## File: path/to/file)
  b. The full contents of the file in a code block

## Usage Guidelines
- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.

## Notes
- Some files may have been excluded based on .gitignore rules and Repomix's configuration
- Binary files are not included in this packed representation. Please refer to the Repository Structure section for a complete list of file paths, including binary files
- Only files matching these patterns are included: src/components/npm-stats/**, src/routes/stats/**, src/blog/npm-stats-the-right-way.md
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)

# Directory Structure
````
src/
  blog/
    npm-stats-the-right-way.md
  components/
    npm-stats/
      BaselineSection.tsx
      binning.ts
      ChartControls.tsx
      ColorPickerPopover.tsx
      DisabledChartActions.tsx
      index.ts
      LatestBucketNavigator.tsx
      npmChartTooltip.ts
      npmQueryOptions.ts
      NPMStatsChart.tsx
      NPMSummary.tsx
      PackagePills.tsx
      PackageSearch.tsx
      PopularComparisons.tsx
      Resizable.tsx
      shared.ts
      StatsTable.tsx
      svgDimensions.ts
  routes/
    stats/
      npm/
        -comparisons.ts
        -utils.ts
        $packages.tsx
        embed.tsx
        index.tsx
      index.tsx
````

# Files

## File: src/blog/npm-stats-the-right-way.md
````markdown
---
title: 'How We Track Billions of Downloads: The NPM Stats API Deep Dive'
published: 2025-12-02
excerpt: When you're tracking download stats for 200+ packages with over 4 billion total downloads, you learn a few things about NPM's API. Some lessons are documented. Others you discover the hard way.
authors:
  - Tanner Linsley
---

When you're tracking download stats for an ecosystem of 200+ packages that have been downloaded over 4 billion times, you learn a few things about NPM's download counts API. Some of those lessons are documented. Others you discover the hard way.

This post is about one of those hard-learned lessons: **why we can't just ask NPM for all-time download stats in a single request, and why the approach matters more than you'd think.**

---

## The Two Faces of NPM Stats

NPM offers two endpoints for download statistics:

**The Point Endpoint** (`/downloads/point/{period}/{package}`)

```json
{
  "downloads": 585291892,
  "start": "2024-06-06",
  "end": "2025-12-06",
  "package": "@tanstack/react-query"
}
```

This gives you a single aggregate number. Clean, simple, exactly what you want.

**The Range Endpoint** (`/downloads/range/{period}/{package}`)

```json
{
  "downloads": [
    { "day": "2024-06-06", "downloads": 1904088 },
    { "day": "2024-06-07", "downloads": 1847293 },
    ...
  ],
  "start": "2024-06-06",
  "end": "2025-12-06",
  "package": "@tanstack/react-query"
}
```

This gives you day-by-day breakdowns. More data, but you have to sum it yourself if you want totals.

On the surface, these should return the same numbers. The range endpoint just has more detail, right?

Not quite.

---

## The 18-Month Wall

Here's what the docs say: **both endpoints are limited to 18 months of historical data for standard queries.**

But here's what the docs don't emphasize: when you request more than 18 months, **the API silently truncates your results.**

Let me show you what I mean.

---

## The Experiment

I built a script to test this. Simple premise: query the same packages with both endpoints across different time ranges, and see what happens.

```javascript
// Query @tanstack/react-query for different time periods
const periods = [
  'last-week', // 7 days
  'last-month', // 30 days
  '2024-12-06:2025-12-06', // 12 months
  '2024-06-06:2025-12-06', // 18 months
  '2023-12-07:2025-12-06', // 24 months
  '2015-01-10:2025-12-06', // All-time
]
```

For short periods (7 days, 30 days, 12 months), everything worked perfectly. Both endpoints returned identical data:

```
Last 7 days:
  Point:  13,085,419
  Range:  13,085,419 (7 days)
  ✅ Difference: 0.00%

Last 30 days:
  Point:  54,052,299
  Range:  54,052,299 (30 days)
  ✅ Difference: 0.00%

12 months:
  Point:  479,463,656
  Range:  479,463,656 (366 days)
  ✅ Difference: 0.00%
```

Great! The endpoints match. Time to query all-time stats.

---

## The Plot Twist

Here's where things get interesting:

```
18 months:
  Point:  585,291,892
  Range:  585,291,892 (549 days)
  ✅ Difference: 0.00%

24 months (beyond limit):
  Point:  585,291,892
  Range:  585,291,892 (549 days)
  ✅ Difference: 0.00%

All-time (from 2015):
  Point:  585,291,892
  Range:  585,291,892 (549 days)
  ✅ Difference: 0.00%
```

Notice something? **The numbers are identical for 18 months, 24 months, and all-time.**

Same downloads. Same number of days (549). **Both endpoints are silently capped at roughly 18 months**, returning exactly 549 days of data no matter what date range you request.

This means:

- Requesting "all downloads since 2015" gives you 18 months of data
- The API doesn't tell you it truncated your request
- Both endpoints fail the same way

TanStack Query has been around since 2019. React has been around since 2013. If we just asked NPM for "all-time" stats, we'd be missing **years** of download history.

---

## Proof: The Real Numbers

To validate this, I ran a second test comparing single requests vs properly chunked requests:

**Single Request** (2019-10-25 to today):

```
Downloads: 585,291,892
Days:      549 days
```

**Chunked Requests** (same period, 5 chunks of ~500 days each):

```
Chunk 1 (2019-10-25 → 2021-03-08): 0 downloads
Chunk 2 (2021-03-09 → 2022-07-22): 0 downloads
Chunk 3 (2022-07-23 → 2023-12-05): 86,135,448 downloads
Chunk 4 (2023-12-06 → 2025-04-19): 284,835,067 downloads
Chunk 5 (2025-04-20 → 2025-12-06): 373,366,977 downloads

Total:     744,337,492
Days:      2,235 days
```

**The difference? 159 million downloads.** That's **27% of the data** completely missing from the single request approach.

You can verify this yourself using tools like [npm-stat.com](https://npm-stat.com), which properly implements chunking and shows ~744M downloads for TanStack Query - matching our chunked approach, not the naive single-request number.

---

## Why This Matters

When you're tracking growth for an open source ecosystem, accuracy matters. Not for vanity metrics, but because:

1. **Sponsorship decisions** are made based on real adoption numbers
2. **Contributors want to see impact** from their work
3. **Companies evaluating libraries** look at download trends

If we naively queried for all-time stats, we'd be reporting 585 million downloads for TanStack Query. The real number? **744 million**. That's 159 million downloads (27%) missing.

For the entire TanStack ecosystem with 200+ packages? We'd be off by billions.

---

## The Right Way: Chunked Requests

The solution is to break time into chunks and request each period separately:

```typescript
async function fetchAllTimeDownloads(packageName: string, createdDate: string) {
  const chunks = []
  const maxChunkDays = 500 // Stay under 18-month limit

  let currentDate = new Date(createdDate)
  const today = new Date()

  while (currentDate < today) {
    const chunkEnd = new Date(currentDate)
    chunkEnd.setDate(chunkEnd.getDate() + maxChunkDays)

    if (chunkEnd > today) {
      chunkEnd = today
    }

    const from = formatDate(currentDate)
    const to = formatDate(chunkEnd)

    const url = `https://api.npmjs.org/downloads/range/${from}:${to}/${packageName}`
    const data = await fetchWithRetry(url)

    chunks.push(data)
    currentDate = chunkEnd

    // Small delay to avoid rate limiting
    await sleep(200)
  }

  // Sum all chunks
  return chunks.reduce((total, chunk) => {
    return total + chunk.downloads.reduce((sum, day) => sum + day.downloads, 0)
  }, 0)
}
```

This approach:

- Breaks the timeline into ~17-month chunks (staying safely under the limit)
- Fetches each chunk sequentially to avoid rate limiting
- Sums the results to get true all-time totals
- Includes retry logic for reliability

It's more work, but it's the only way to get accurate historical data.

---

## Point vs Range: Which One?

After all this, you might wonder: should you use `/point/` or `/range/`?

**For all-time stats, use `/range/` with chunking.** Here's why:

1. **They return the same totals** (when you sum the daily breakdowns from range)
2. **Range gives you daily granularity** for trend analysis
3. **Range is what you need for chunking anyway** (you have to sum across chunks)
4. **Both are limited to 18 months**, so there's no advantage to point

The point endpoint is useful for quick spot checks or when you only need recent data. But for building a real stats system, range is the way to go.

---

## Our Implementation

At TanStack, we've built a sophisticated stats system that handles this properly:

- **Automatic chunking** for packages created before 18 months ago
- **Rate limit handling** with exponential backoff
- **Concurrent processing** of 8 packages at a time (to balance speed and API limits)
- **Database caching** with 24-hour TTL to avoid hammering NPM
- **Scheduled refreshes** every 6 hours via Netlify functions
- **Growth rate calculation** from the most recent 7 days for live animations

The full implementation is in [`src/utils/stats.functions.ts`](https://github.com/TanStack/tanstack.com/blob/main/src/utils/stats.functions.ts) if you want to see how we handle the details.

### Library-Level Aggregation

One interesting aspect of our system is how we track stats at the library level. Each TanStack library (Query, Table, Router, etc.) maps to a GitHub repository, and we aggregate downloads for **all npm packages** published from that repo.

This includes:

1. **Scoped packages**: Like `@tanstack/react-query`, `@tanstack/query-core`
2. **Legacy packages**: Like `react-query` (the pre-rebrand name)
3. **Addon packages**: Like `@tanstack/react-query-devtools`, `@tanstack/react-query-persist-client`

For example, TanStack Query's library stats include:

```typescript
// From src/libraries/query.tsx
{
  repo: 'tanstack/query',
  legacyPackages: ['react-query']
}
```

This means our library metrics sum up all related packages, which can include addons and dependencies that might also depend on the core package. This **could inflate library-level numbers** since some downloads might be for packages that themselves depend on other packages in the same library.

We know this. We're keeping it simple for now.

The alternative would be dependency analysis and deduplication - figuring out which packages depend on each other and avoiding double-counting. That's a project for another day. For now, the simple aggregation gives us a reasonable approximation of ecosystem reach, even if it's not perfectly precise.

What matters is consistency: we track the same way over time, so trends and growth rates remain meaningful.

---

## The Results

This approach lets us accurately track downloads across **203 packages** (199 scoped @tanstack/\* + 4 legacy packages), maintaining historical accuracy going back to 2015.

The stats you see on [tanstack.com](/stats) aren't guesses or estimates. They're the sum of thousands of individual API calls, properly chunked, cached, and aggregated.

When we say TanStack has been downloaded over 4 billion times, that number is real. And it's growing by millions every day.

---

## Key Takeaways

If you're building a system to track NPM download stats:

1. **Never trust a single "all-time" request** - it's capped at 18 months
2. **Use the `/range/` endpoint with chunking** for historical accuracy
3. **Implement retry logic** - rate limiting will happen
4. **Cache aggressively** - NPM's data doesn't update instantly
5. **Test your assumptions** - build experiments to verify behavior

The NPM download counts API is powerful, but it has sharp edges. Understanding these limitations is the difference between showing users vanity metrics and giving them real data.

---

## Why This Matters to TanStack

We care about this because **transparency matters**. When we show download stats, we want them to be accurate. When we talk about growth, we want it to be real.

The same principle applies to our libraries. We don't hide complexity behind magic. We build tools that are powerful when you need them to be, and simple when you don't.

That's the TanStack way.

---

**Want to dive deeper into how we build TanStack?** [Join our Discord](https://tlinz.com/discord) where we talk about architecture, API design, and the technical decisions behind the ecosystem.

**Using TanStack and want to support the work?** [Check out our sponsors and partners page](/partners). Every contribution helps us keep building open source the right way.
````

## File: src/components/npm-stats/BaselineSection.tsx
````typescript
import * as React from 'react'
import {
  PushPinIcon,
  PushPinSlashIcon,
  PlusIcon,
  XIcon,
  CaretDownIcon,
  EyeIcon,
  EyeSlashIcon,
} from '@phosphor-icons/react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@radix-ui/react-dropdown-menu'
import { twMerge } from 'tailwind-merge'
import { Tooltip } from '~/components/Tooltip'
import {
  Dialog,
  DialogBody,
  DialogContent,
  DialogHeader,
} from '~/components/ds/ui'
import { PackageSearch } from './PackageSearch'
import { getBaselineDisplayName, type PackageGroup } from './shared'
import type { BaselinePreset } from '~/routes/stats/npm/-comparisons'

export const BASELINE_LINE_COLOR = '#3b82f6'

export type BaselineSectionProps = {
  packageGroups: PackageGroup[]
  presets: BaselinePreset[]
  canToggleShowBaseline: boolean
  normalizeBaseline: boolean
  showBaseline: boolean
  onToggleNormalizeBaseline: () => void
  onToggleShowBaseline: () => void
  onAddBaseline: (packageName: string, color?: string) => void
  onApplyPreset: (preset: BaselinePreset) => void
  onClearBaselines: () => void
}

export function BaselineSection({
  packageGroups,
  presets,
  canToggleShowBaseline,
  normalizeBaseline,
  showBaseline,
  onToggleNormalizeBaseline,
  onToggleShowBaseline,
  onAddBaseline,
  onApplyPreset,
  onClearBaselines,
}: BaselineSectionProps) {
  const baselineGroups = packageGroups.filter((pg) => pg.baseline)
  const [showSearch, setShowSearch] = React.useState(false)
  const baselineDisplayName = getBaselineDisplayName(baselineGroups)
  const hasBaselines = baselineGroups.length > 0
  const normalizeActive = hasBaselines && normalizeBaseline

  const labelButton = (
    <Tooltip
      content={
        !hasBaselines
          ? 'Add baseline packages to enable normalization'
          : normalizeBaseline
            ? 'Disable normalization (stop dividing other lines by baseline)'
            : 'Enable normalization (divide other lines by combined baseline)'
      }
    >
      <button
        onClick={onToggleNormalizeBaseline}
        disabled={!hasBaselines}
        className={twMerge(
          'flex items-center gap-1 font-semibold',
          normalizeActive
            ? 'text-blue-600 dark:text-blue-400 hover:text-blue-700 dark:hover:text-blue-300'
            : 'text-gray-400 dark:text-gray-500 hover:text-gray-600 dark:hover:text-gray-300',
          !hasBaselines && 'cursor-default opacity-60',
        )}
      >
        {normalizeActive ? (
          <PushPinIcon className="w-3 h-3" />
        ) : (
          <PushPinSlashIcon className="w-3 h-3" />
        )}
        <span>Baseline</span>
      </button>
    </Tooltip>
  )

  const isBaselineShown = canToggleShowBaseline && showBaseline
  const baselineName = (
    <span className={twMerge('font-medium', !isBaselineShown && 'opacity-70')}>
      {baselineDisplayName}
    </span>
  )

  const chip = hasBaselines && (
    <div
      className="flex items-center gap-1.5 rounded px-1.5 py-0.5 text-xs text-blue-900 dark:text-blue-100"
      style={{ backgroundColor: `${BASELINE_LINE_COLOR}25` }}
    >
      {canToggleShowBaseline ? (
        <Tooltip
          content={
            isBaselineShown
              ? 'Hide combined baseline line on chart'
              : 'Show combined baseline line on chart'
          }
        >
          <button
            onClick={onToggleShowBaseline}
            className="flex items-center gap-1.5 hover:brightness-110"
          >
            {isBaselineShown ? (
              <EyeIcon className="w-3 h-3 text-blue-600 dark:text-blue-300" />
            ) : (
              <EyeSlashIcon className="w-3 h-3 text-gray-400 dark:text-gray-500" />
            )}
            {baselineName}
          </button>
        </Tooltip>
      ) : (
        <div className="flex items-center gap-1.5">
          <EyeSlashIcon className="w-3 h-3 text-gray-400 dark:text-gray-500" />
          {baselineName}
        </div>
      )}
      <Tooltip content="Clear all baselines">
        <button
          onClick={onClearBaselines}
          className="text-gray-500 hover:text-red-500"
          aria-label="Clear all baselines"
        >
          <XIcon className="w-3 h-3" />
        </button>
      </Tooltip>
    </div>
  )

  const addButton = (
    <Tooltip content="Search for a package to add as baseline">
      <button
        onClick={() => setShowSearch(true)}
        className="flex items-center gap-1 px-1.5 py-0.5 text-xs rounded
          text-blue-700 dark:text-blue-300 hover:bg-blue-500/10 font-medium"
      >
        <PlusIcon className="w-3 h-3" />
        Add
      </button>
    </Tooltip>
  )

  const presetsMenu = (
    <DropdownMenu>
      <Tooltip content="Apply a curated baseline preset">
        <DropdownMenuTrigger asChild>
          <button
            className="flex items-center gap-0.5 px-1.5 py-0.5 text-xs rounded
              text-blue-700 dark:text-blue-300 hover:bg-blue-500/10 font-medium"
          >
            Presets
            <CaretDownIcon className="w-3 h-3" />
          </button>
        </DropdownMenuTrigger>
      </Tooltip>
      <DropdownMenuContent
        className="min-w-[320px] max-w-[400px] bg-white dark:bg-gray-800 rounded-lg shadow-lg p-2 z-50 border border-gray-200 dark:border-gray-700"
        sideOffset={5}
        align="start"
      >
        <div className="px-2 pt-1 pb-0.5 text-xs font-semibold text-gray-500 dark:text-gray-400 uppercase tracking-wide">
          Single Package
        </div>
        {presets
          .filter((p) => p.category === 'single')
          .map((preset) => (
            <DropdownMenuItem
              key={preset.id}
              onSelect={(e) => {
                e.preventDefault()
                onApplyPreset(preset)
              }}
              className="w-full px-2 py-1.5 text-left text-sm rounded hover:bg-gray-500/10 outline-none cursor-pointer flex items-center gap-2"
            >
              <div
                className="w-2.5 h-2.5 rounded"
                style={{
                  backgroundColor: preset.packages[0]?.color ?? 'currentColor',
                }}
              />
              <span className="font-medium text-gray-900 dark:text-gray-100">
                {preset.title}
              </span>
            </DropdownMenuItem>
          ))}

        <div className="h-px bg-gray-500/20 my-1.5" />

        <div className="px-2 pt-1 pb-0.5 text-xs font-semibold text-gray-500 dark:text-gray-400 uppercase tracking-wide">
          Equal-Weighted Index
        </div>
        {presets
          .filter((p) => p.category === 'index')
          .map((preset) => (
            <DropdownMenuItem
              key={preset.id}
              onSelect={(e) => {
                e.preventDefault()
                onApplyPreset(preset)
              }}
              className="w-full px-2 py-2 text-left text-sm rounded hover:bg-gray-500/10 outline-none cursor-pointer"
            >
              <div className="font-medium text-gray-900 dark:text-gray-100">
                {preset.title}
              </div>
              <div className="text-xs text-gray-500 dark:text-gray-400 mt-0.5">
                {preset.description}
              </div>
              <div className="flex flex-wrap gap-1 mt-1.5">
                {preset.packages.map((p) => (
                  <div
                    key={p.name}
                    className="flex items-center gap-1 text-[10px] font-mono text-gray-600 dark:text-gray-300"
                  >
                    <div
                      className="w-2 h-2 rounded"
                      style={{ backgroundColor: p.color ?? 'currentColor' }}
                    />
                    {p.name}
                  </div>
                ))}
              </div>
            </DropdownMenuItem>
          ))}
      </DropdownMenuContent>
    </DropdownMenu>
  )

  return (
    <>
      <div className="flex flex-wrap items-center gap-x-2 gap-y-1 text-xs">
        {labelButton}

        {!hasBaselines ? (
          <span className="text-gray-500 dark:text-gray-400 italic">none</span>
        ) : (
          chip
        )}

        <div className="flex items-center gap-0.5">
          {addButton}
          {presetsMenu}
        </div>
      </div>

      <Dialog open={showSearch} onOpenChange={setShowSearch}>
        <DialogContent size="md">
          <DialogHeader
            title={
              <span className="flex items-center gap-2">
                <PushPinIcon className="w-4 h-4 text-icon-accent" />
                Add baseline package
              </span>
            }
          />
          <DialogBody className="pb-6">
            <PackageSearch
              onSelect={(pkg) => {
                onAddBaseline(pkg)
                setShowSearch(false)
              }}
              placeholder="Search for baseline package..."
              // eslint-disable-next-line jsx-a11y/no-autofocus
              autoFocus={true}
            />
          </DialogBody>
        </DialogContent>
      </Dialog>
    </>
  )
}
````

## File: src/components/npm-stats/binning.ts
````typescript
import * as d3 from 'd3'

import type { BinType, TimeRange } from './shared'

export const NPM_STATS_START_DATE = d3.utcDay(new Date('2015-01-10'))

export const binningOptionsByType = {
  yearly: {
    label: 'Yearly',
    value: 'yearly',
    single: 'year',
    bin: d3.utcYear,
  },
  monthly: {
    label: 'Monthly',
    value: 'monthly',
    single: 'month',
    bin: d3.utcMonth,
  },
  weekly: {
    label: 'Weekly',
    value: 'weekly',
    single: 'week',
    bin: d3.utcWeek,
  },
  daily: {
    label: 'Daily',
    value: 'daily',
    single: 'day',
    bin: d3.utcDay,
  },
} as const satisfies Record<
  BinType,
  {
    label: string
    value: BinType
    single: string
    bin: (date: Date) => Date
  }
>

export function getUtcToday() {
  const today = d3.utcDay(new Date())
  today.setHours(0, 0, 0, 0)
  return today
}

export function getUtcDay(date: Date) {
  return d3.utcDay(date)
}

export function getHistoryStartDate(range: TimeRange, now = getUtcToday()) {
  switch (range) {
    case '7-days':
      return d3.utcDay.offset(now, -7)
    case '30-days':
      return d3.utcDay.offset(now, -30)
    case '90-days':
      return d3.utcDay.offset(now, -90)
    case '180-days':
      return d3.utcDay.offset(now, -180)
    case '365-days':
      return d3.utcDay.offset(now, -365)
    case '730-days':
      return d3.utcDay.offset(now, -730)
    case '1825-days':
      return d3.utcDay.offset(now, -1825)
    case 'all-time':
      return NPM_STATS_START_DATE
  }
}

export function formatNpmStatsDate(date: Date) {
  return date.toISOString().split('T')[0]
}

export function getLatestBucketWindow({
  binType,
  bucketOffset,
  now = getUtcToday(),
}: {
  binType: BinType
  bucketOffset: number
  now?: Date
}) {
  const binUnit = binningOptionsByType[binType].bin
  const currentBucketStart = binUnit.floor(now)
  const clampedOffset = Math.min(bucketOffset, 0)
  const startDate = binUnit.offset(currentBucketStart, clampedOffset)
  const nextBucketStart = binUnit.offset(startDate, 1)
  const lastBucketDate = d3.utcDay.offset(nextBucketStart, -1)
  const endDate = lastBucketDate > now ? now : lastBucketDate

  return {
    startDate,
    endDate,
    nextBucketStart,
  }
}

export function getLatestBucketOffsetBounds({
  binType,
  range,
  now = getUtcToday(),
}: {
  binType: BinType
  range?: TimeRange
  now?: Date
}) {
  const binUnit = binningOptionsByType[binType].bin
  const currentBucketStart = binUnit.floor(now)
  const rangeStartDate = range ? getHistoryStartDate(range, now) : undefined
  const firstAvailableDate =
    rangeStartDate && rangeStartDate > NPM_STATS_START_DATE
      ? rangeStartDate
      : NPM_STATS_START_DATE
  const firstBucketStart = binUnit.floor(firstAvailableDate)
  const minOffset = -binUnit.count(firstBucketStart, currentBucketStart)

  return {
    minOffset,
    maxOffset: 0,
  }
}

export function formatLatestBucketLabel({
  binType,
  startDate,
  endDate,
}: {
  binType: BinType
  startDate: Date
  endDate: Date
}) {
  switch (binType) {
    case 'yearly':
      return startDate.toLocaleDateString('en-US', {
        timeZone: 'UTC',
        year: 'numeric',
      })
    case 'monthly':
      return startDate.toLocaleDateString('en-US', {
        timeZone: 'UTC',
        month: 'long',
        year: 'numeric',
      })
    case 'weekly':
      return `${startDate.toLocaleDateString('en-US', {
        timeZone: 'UTC',
        month: 'short',
        day: 'numeric',
      })} - ${endDate.toLocaleDateString('en-US', {
        timeZone: 'UTC',
        month: 'short',
        day: 'numeric',
        year: 'numeric',
      })}`
    case 'daily':
      return startDate.toLocaleDateString('en-US', {
        timeZone: 'UTC',
        month: 'long',
        day: 'numeric',
        year: 'numeric',
      })
  }
}

export function getLatestBucketOffsetOptions({
  binType,
  range,
  now = getUtcToday(),
}: {
  binType: BinType
  range?: TimeRange
  now?: Date
}) {
  const bounds = getLatestBucketOffsetBounds({ binType, range, now })
  const options: Array<{ offset: number; label: string }> = []

  for (
    let bucketOffset = bounds.maxOffset;
    bucketOffset >= bounds.minOffset;
    bucketOffset--
  ) {
    const window = getLatestBucketWindow({ binType, bucketOffset, now })

    options.push({
      offset: bucketOffset,
      label: formatLatestBucketLabel({
        binType,
        startDate: window.startDate,
        endDate: window.endDate,
      }),
    })
  }

  return options
}
````

## File: src/components/npm-stats/ChartControls.tsx
````typescript
import * as React from 'react'
import { twMerge } from 'tailwind-merge'
import {
  SortDescendingIcon,
  ChartLineUpIcon,
  ChartBarIcon,
  ChartLineIcon,
  ClockIcon,
  ColumnsIcon,
  ClockCounterClockwiseIcon,
  StackIcon,
  RowsIcon,
  WavesIcon,
  type Icon,
} from '@phosphor-icons/react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@radix-ui/react-dropdown-menu'
import { Tooltip } from '~/components/Tooltip'
import {
  type TimeRange,
  type BinType,
  type ChartType,
  type TransformMode,
  type ShowDataMode,
  type ViewMode,
  type LatestBarSort,
  type BarOrientation,
  timeRanges,
  binningOptions,
  historyChartTypes,
  latestChartTypes,
  isBinningOptionValidForRange,
} from './shared'

const dropdownButtonStyles = {
  base: 'flex h-6 items-center gap-0.5 rounded bg-gray-500/10 px-1.5 py-0.5 text-xs leading-none',
  active: 'bg-gray-500/20',
} as const

const dropdownContentStyles =
  'min-w-[180px] rounded-md bg-white p-1.5 shadow-lg dark:bg-gray-800 z-50'
const dropdownHeaderStyles =
  'mb-1 flex items-center justify-between px-0.5 text-xs font-medium'
const dropdownItemStyles =
  'flex w-full cursor-pointer items-center gap-1.5 rounded px-1.5 py-1 text-left text-xs outline-none hover:bg-gray-500/20 data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500'
const segmentedControlStyles =
  'flex h-6 items-center rounded bg-gray-500/10 p-0.5 text-xs leading-none'
const segmentedButtonStyles = 'flex h-5 items-center gap-0.5 rounded px-1.5'
const iconSegmentedButtonStyles =
  'flex size-5 items-center justify-center rounded'

const chartTypeIcons = {
  line: ChartLineIcon,
  stacked: StackIcon,
  'stacked-area': ChartLineUpIcon,
  'stacked-stream': WavesIcon,
  bar: ChartBarIcon,
  'stacked-bar': ChartBarIcon,
} as const satisfies Partial<Record<ChartType, Icon>>

const transformOptions = [
  { value: 'none', label: 'Actual Values' },
  { value: 'normalize-y', label: 'Relative Change' },
] as const satisfies ReadonlyArray<{
  value: TransformMode
  label: string
}>

const showDataModeOptions = [
  { value: 'all', label: 'All Data' },
  { value: 'complete', label: 'Hide Partial Data' },
] as const

export type ChartControlsProps = {
  viewMode: ViewMode
  chartType: ChartType
  range: TimeRange
  binType: BinType
  transform: TransformMode
  showDataMode: ShowDataMode
  barSort: LatestBarSort
  barOrientation: BarOrientation
  onViewModeChange: (mode: ViewMode) => void
  onChartTypeChange: (chartType: ChartType) => void
  onRangeChange: (range: TimeRange) => void
  onBinTypeChange: (binType: BinType) => void
  onTransformChange: (transform: TransformMode) => void
  onShowDataModeChange: (mode: ShowDataMode) => void
  onBarSortChange: (barSort: LatestBarSort) => void
  onBarOrientationChange: (barOrientation: BarOrientation) => void
}

export function ChartControls({
  viewMode,
  chartType,
  range,
  binType,
  transform,
  showDataMode,
  barSort,
  barOrientation,
  onViewModeChange,
  onChartTypeChange,
  onRangeChange,
  onBinTypeChange,
  onTransformChange,
  onShowDataModeChange,
  onBarSortChange,
  onBarOrientationChange,
}: ChartControlsProps) {
  const chartTypeOptions =
    viewMode === 'history' ? historyChartTypes : latestChartTypes
  const canUseTransform = viewMode === 'history' && chartType === 'line'
  const showDataModeDisabled =
    viewMode === 'latest' || transform === 'normalize-y'
  const showBarSort =
    viewMode === 'latest' &&
    (chartType === 'bar' || chartType === 'stacked-bar')
  const showBarOrientation = showBarSort

  return (
    <>
      <div className={segmentedControlStyles}>
        <Tooltip content="Show the full timeline">
          <button
            onClick={() => onViewModeChange('history')}
            className={twMerge(
              segmentedButtonStyles,
              viewMode === 'history'
                ? 'bg-white text-blue-600 shadow-sm dark:bg-gray-900 dark:text-blue-400'
                : 'text-gray-600 hover:text-gray-900 dark:text-gray-400 dark:hover:text-gray-100',
            )}
          >
            <ClockCounterClockwiseIcon className="size-3" />
            Timeline
          </button>
        </Tooltip>
        <Tooltip content="Show one selected bucket snapshot">
          <button
            onClick={() => onViewModeChange('latest')}
            className={twMerge(
              segmentedButtonStyles,
              viewMode === 'latest'
                ? 'bg-white text-blue-600 shadow-sm dark:bg-gray-900 dark:text-blue-400'
                : 'text-gray-600 hover:text-gray-900 dark:text-gray-400 dark:hover:text-gray-100',
            )}
          >
            <ClockIcon className="size-3" />
            Snapshot
          </button>
        </Tooltip>
      </div>

      {/* Time Range */}
      <DropdownMenu>
        <Tooltip content="Select time range">
          <DropdownMenuTrigger asChild>
            <button className={twMerge(dropdownButtonStyles.base)}>
              {timeRanges.find((r) => r.value === range)?.label}
            </button>
          </DropdownMenuTrigger>
        </Tooltip>
        <DropdownMenuContent className={dropdownContentStyles}>
          <div className={dropdownHeaderStyles}>
            <span>Time Range</span>
          </div>
          {timeRanges.map(({ value, label }) => (
            <DropdownMenuItem
              key={value}
              onSelect={() => onRangeChange(value)}
              className={twMerge(
                dropdownItemStyles,
                value === range ? 'text-blue-500 bg-blue-500/10' : '',
                'data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500',
              )}
            >
              {label}
            </DropdownMenuItem>
          ))}
        </DropdownMenuContent>
      </DropdownMenu>

      {/* Binning Interval */}
      <DropdownMenu>
        <Tooltip content="Select binning interval">
          <DropdownMenuTrigger asChild>
            <button
              className={twMerge(
                dropdownButtonStyles.base,
                binType !== 'weekly' && dropdownButtonStyles.active,
              )}
            >
              {binningOptions.find((b) => b.value === binType)?.label}
            </button>
          </DropdownMenuTrigger>
        </Tooltip>
        <DropdownMenuContent className={dropdownContentStyles}>
          <div className={dropdownHeaderStyles}>
            <span>Binning Interval</span>
          </div>
          {binningOptions.map(({ label, value }) => (
            <DropdownMenuItem
              key={value}
              onSelect={() => onBinTypeChange(value)}
              disabled={
                viewMode === 'history' &&
                !isBinningOptionValidForRange(range, value)
              }
              className={twMerge(
                dropdownItemStyles,
                binType === value ? 'text-blue-500 bg-blue-500/10' : '',
                'data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500',
                viewMode === 'history' &&
                  !isBinningOptionValidForRange(range, value)
                  ? 'opacity-50 cursor-not-allowed'
                  : '',
              )}
            >
              {label}
            </DropdownMenuItem>
          ))}
        </DropdownMenuContent>
      </DropdownMenu>

      {/* Chart Type */}
      <div aria-label="Chart type" className={segmentedControlStyles}>
        {chartTypeOptions.map(({ value, label }) => {
          const Icon = chartTypeIcons[value] ?? ChartBarIcon

          return (
            <Tooltip content={label} key={value}>
              <button
                aria-label={`${label} chart`}
                aria-pressed={chartType === value}
                className={twMerge(
                  iconSegmentedButtonStyles,
                  chartType === value
                    ? 'bg-white text-blue-600 shadow-sm dark:bg-gray-900 dark:text-blue-400'
                    : 'text-gray-600 hover:text-gray-900 dark:text-gray-400 dark:hover:text-gray-100',
                )}
                onClick={() => onChartTypeChange(value)}
                type="button"
              >
                <Icon className="size-3.5" />
              </button>
            </Tooltip>
          )
        })}
      </div>

      {showBarOrientation ? (
        <div className={segmentedControlStyles}>
          <Tooltip content="Use vertical bars">
            <button
              onClick={() => onBarOrientationChange('vertical')}
              className={twMerge(
                segmentedButtonStyles,
                barOrientation === 'vertical'
                  ? 'bg-white text-blue-600 shadow-sm dark:bg-gray-900 dark:text-blue-400'
                  : 'text-gray-600 hover:text-gray-900 dark:text-gray-400 dark:hover:text-gray-100',
              )}
            >
              <ColumnsIcon className="size-3" />
              Vertical
            </button>
          </Tooltip>
          <Tooltip content="Use horizontal bars">
            <button
              onClick={() => onBarOrientationChange('horizontal')}
              className={twMerge(
                segmentedButtonStyles,
                barOrientation === 'horizontal'
                  ? 'bg-white text-blue-600 shadow-sm dark:bg-gray-900 dark:text-blue-400'
                  : 'text-gray-600 hover:text-gray-900 dark:text-gray-400 dark:hover:text-gray-100',
              )}
            >
              <RowsIcon className="size-3" />
              Horizontal
            </button>
          </Tooltip>
        </div>
      ) : null}

      {showBarSort ? (
        <Tooltip
          content={
            barSort === 'value'
              ? 'Sort bars by series name'
              : 'Sort bars by value'
          }
        >
          <button
            className={twMerge(
              dropdownButtonStyles.base,
              barSort === 'value' && dropdownButtonStyles.active,
            )}
            onClick={() =>
              onBarSortChange(barSort === 'value' ? 'name' : 'value')
            }
          >
            <SortDescendingIcon className="size-3" />
            {barSort === 'value' ? 'Value Sort' : 'Name Sort'}
          </button>
        </Tooltip>
      ) : null}

      {/* Y-Axis Transform */}
      <DropdownMenu>
        <Tooltip
          content={
            canUseTransform
              ? "Transform the Y-axis to show relative changes between packages. 'None' shows actual download numbers, while 'Normalize Y' shows percentage changes relative to the first data point."
              : 'Relative change is only available for history line charts'
          }
        >
          <DropdownMenuTrigger asChild>
            <button
              className={twMerge(
                dropdownButtonStyles.base,
                transform !== 'none' && dropdownButtonStyles.active,
                !canUseTransform && 'opacity-50 cursor-not-allowed',
              )}
              disabled={!canUseTransform}
            >
              {transformOptions.find((opt) => opt.value === transform)?.label}
            </button>
          </DropdownMenuTrigger>
        </Tooltip>
        <DropdownMenuContent className={dropdownContentStyles}>
          <div className={dropdownHeaderStyles}>
            <span>Y-Axis Transform</span>
          </div>
          {transformOptions.map(({ value, label }) => (
            <DropdownMenuItem
              key={value}
              onSelect={() => onTransformChange(value)}
              disabled={!canUseTransform}
              className={twMerge(
                dropdownItemStyles,
                transform === value ? 'text-blue-500 bg-blue-500/10' : '',
                'data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500',
                !canUseTransform ? 'opacity-50 cursor-not-allowed' : '',
              )}
            >
              {label}
            </DropdownMenuItem>
          ))}
        </DropdownMenuContent>
      </DropdownMenu>

      {/* Show Data Mode */}
      <DropdownMenu>
        <Tooltip
          content={
            viewMode === 'latest'
              ? 'Snapshot mode always shows the selected bucket'
              : transform === 'normalize-y'
                ? 'Only complete data is shown when using relative change'
                : 'Control how data is displayed'
          }
        >
          <DropdownMenuTrigger asChild>
            <button
              className={twMerge(
                dropdownButtonStyles.base,
                showDataMode !== 'all' && dropdownButtonStyles.active,
                showDataModeDisabled && 'opacity-50 cursor-not-allowed',
              )}
              disabled={showDataModeDisabled}
            >
              {
                showDataModeOptions.find((opt) => opt.value === showDataMode)
                  ?.label
              }
            </button>
          </DropdownMenuTrigger>
        </Tooltip>
        <DropdownMenuContent className={dropdownContentStyles}>
          <div className={dropdownHeaderStyles}>
            <span>Data Display Mode</span>
          </div>
          {showDataModeOptions.map(({ value, label }) => (
            <DropdownMenuItem
              key={value}
              onSelect={() => onShowDataModeChange(value)}
              disabled={showDataModeDisabled}
              className={twMerge(
                dropdownItemStyles,
                showDataMode === value ? 'text-blue-500 bg-blue-500/10' : '',
                'data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500',
                showDataModeDisabled ? 'opacity-50 cursor-not-allowed' : '',
              )}
            >
              {label}
            </DropdownMenuItem>
          ))}
        </DropdownMenuContent>
      </DropdownMenu>
    </>
  )
}
````

## File: src/components/npm-stats/ColorPickerPopover.tsx
````typescript
import * as React from 'react'
import { createPortal } from 'react-dom'
import { HexColorPicker } from 'react-colorful'

export interface ColorPickerPopoverProps {
  packageName: string
  position: { x: number; y: number }
  currentColor: string
  onColorChange: (packageName: string, color: string) => void
  onReset: (packageName: string) => void
  onClose: () => void
}

/**
 * A popover component for picking package colors.
 * Shows a hex color picker with Reset and Done buttons.
 */
export function ColorPickerPopover({
  packageName,
  position,
  currentColor,
  onColorChange,
  onReset,
  onClose,
}: ColorPickerPopoverProps) {
  if (typeof document === 'undefined') return null

  return createPortal(
    <div
      className="fixed z-50 bg-white dark:bg-gray-800 rounded-lg shadow-lg p-2"
      style={{
        left: position.x,
        top: position.y,
      }}
    >
      <div className="flex justify-between items-center mb-2">
        <span className="text-sm font-medium">Pick a color</span>
      </div>
      <HexColorPicker
        color={currentColor}
        onChange={(color: string) => onColorChange(packageName, color)}
      />
      <div className="flex justify-between mt-2">
        <button
          onClick={() => {
            onReset(packageName)
            onClose()
          }}
          className="px-2 py-1 text-sm text-gray-500 hover:text-red-500"
        >
          Reset
        </button>
        <button
          onClick={onClose}
          className="px-2 py-1 text-sm text-blue-500 hover:text-blue-600"
        >
          Done
        </button>
      </div>
    </div>,
    document.body,
  )
}
````

## File: src/components/npm-stats/DisabledChartActions.tsx
````typescript
import { DownloadIcon, ListIcon } from '@phosphor-icons/react'
import { Tooltip } from '~/components/Tooltip'

const disabledChartActionButtonStyles =
  'flex size-6 cursor-not-allowed items-center justify-center rounded bg-gray-500/10 text-gray-600 opacity-50 dark:text-gray-400'

export function DisabledChartActions() {
  return (
    <div className="flex items-center gap-1">
      <Tooltip content="Chart export available after render">
        <button
          aria-label="Export chart"
          className={disabledChartActionButtonStyles}
          disabled={true}
          type="button"
        >
          <DownloadIcon className="size-3" />
        </button>
      </Tooltip>
      <Tooltip content="Plot legend available after render">
        <button
          aria-label="Show Plot legend"
          aria-pressed={false}
          className={disabledChartActionButtonStyles}
          disabled={true}
          type="button"
        >
          <ListIcon className="size-3" />
        </button>
      </Tooltip>
    </div>
  )
}
````

## File: src/components/npm-stats/index.ts
````typescript
export { NPMStatsChart, type NPMStatsChartProps } from './NPMStatsChart'

export type {
  NpmQueryData,
  PackageGroup,
  BinType,
  ChartType,
  TransformMode,
  ShowDataMode,
  LatestBarSort,
  BarOrientation,
  TimeRange,
  ViewMode,
} from './shared'

export {
  binTypeSchema,
  chartTypeSchema,
  transformModeSchema,
  showDataModeSchema,
  latestBarSortSchema,
  barOrientationSchema,
  viewModeSchema,
  binningOptions,
  timeRanges,
  defaultRangeBinTypes,
  getPackageColor,
  formatNumber,
  isBinningOptionValidForRange,
  isChartTypeValidForViewMode,
  getDefaultChartTypeForViewMode,
} from './shared'

export { binningOptionsByType } from './binning'

export { PopularComparisons, type ComparisonGroup } from './PopularComparisons'

export { PackageSearch, type PackageSearchProps } from './PackageSearch'

export { Resizable, type ResizableProps } from './Resizable'

export { npmQueryOptions } from './npmQueryOptions'

export {
  ColorPickerPopover,
  type ColorPickerPopoverProps,
} from './ColorPickerPopover'

export { StatsTable, type StatsTableProps } from './StatsTable'

export { NPMSummary } from './NPMSummary'

export {
  LatestBucketNavigator,
  type LatestBucketNavigatorProps,
} from './LatestBucketNavigator'

export { ChartControls, type ChartControlsProps } from './ChartControls'

export {
  PackagePill,
  PackagePills,
  type PackagePillProps,
  type PackagePillsProps,
} from './PackagePills'
````

## File: src/components/npm-stats/LatestBucketNavigator.tsx
````typescript
import * as React from 'react'
import {
  CheckIcon,
  CaretDownIcon,
  CaretLeftIcon,
  CaretRightIcon,
  CaretDoubleLeftIcon,
  CaretDoubleRightIcon,
  DotsThreeIcon,
  PauseIcon,
  PlayIcon,
} from '@phosphor-icons/react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@radix-ui/react-dropdown-menu'
import {
  binningOptionsByType,
  getLatestBucketOffsetBounds,
  getLatestBucketOffsetOptions,
} from './binning'
import {
  defaultPlaybackIntervalMs,
  maxPlaybackIntervalMs,
  type BinType,
  type TimeRange,
} from './shared'

const playbackSpeedOptions = [
  { label: 'Slow', intervalMs: 800 },
  { label: 'Medium', intervalMs: 350 },
  { label: 'Fast', intervalMs: 150 },
  { label: 'Very Fast', intervalMs: 75 },
  { label: 'Very Very Fast', intervalMs: 35 },
] as const

const navigatorButtonStyles =
  'flex size-6 items-center justify-center hover:bg-gray-500/10 disabled:cursor-not-allowed disabled:opacity-40'
const borderedNavigatorButtonStyles = `${navigatorButtonStyles} border-l border-gray-500/15`
const menuItemStyles =
  'flex w-full cursor-pointer items-center gap-1.5 rounded px-1.5 py-1 text-left text-xs outline-none hover:bg-gray-500/20 data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500'

export type LatestBucketNavigatorProps = {
  binType: BinType
  range: TimeRange
  bucketOffset: number
  isLooping: boolean
  isPlaying: boolean
  playbackIntervalMs: number
  onBucketOffsetChange: (bucketOffset: number) => void
  onLoopingChange: (isLooping: boolean) => void
  onPlaybackIntervalChange: (playbackIntervalMs: number) => void
  onPlayingChange: (isPlaying: boolean) => void
}

export function LatestBucketNavigator({
  binType,
  range,
  bucketOffset,
  isLooping,
  isPlaying,
  playbackIntervalMs,
  onBucketOffsetChange,
  onLoopingChange,
  onPlaybackIntervalChange,
  onPlayingChange,
}: LatestBucketNavigatorProps) {
  const bucketSelectId = React.useId()
  const bucketRangeListId = React.useId()
  const playbackIntervalInputId = React.useId()
  const pendingRestartRef = React.useRef(false)
  const [playbackIntervalInput, setPlaybackIntervalInput] = React.useState(
    `${defaultPlaybackIntervalMs}`,
  )
  const bounds = getLatestBucketOffsetBounds({ binType, range })
  const bucketOptions = React.useMemo(
    () => getLatestBucketOffsetOptions({ binType, range }),
    [binType, range],
  )
  const clampedBucketOffset = Math.max(
    bounds.minOffset,
    Math.min(bucketOffset, bounds.maxOffset),
  )
  const binLabel = binningOptionsByType[binType].single
  const canGoBack = clampedBucketOffset > bounds.minOffset
  const canGoForward = clampedBucketOffset < bounds.maxOffset
  const canPlay = bounds.minOffset < bounds.maxOffset

  React.useEffect(() => {
    if (!canPlay && isPlaying) {
      onPlayingChange(false)
    }
  }, [canPlay, isPlaying, onPlayingChange])

  React.useEffect(() => {
    setPlaybackIntervalInput(`${playbackIntervalMs}`)
  }, [playbackIntervalMs])

  React.useEffect(() => {
    pendingRestartRef.current = false
  }, [binType, range])

  React.useEffect(() => {
    if (pendingRestartRef.current && clampedBucketOffset < bounds.maxOffset) {
      pendingRestartRef.current = false
    }
  }, [bounds.maxOffset, clampedBucketOffset])

  React.useEffect(() => {
    if (!isPlaying || !canPlay) return

    if (pendingRestartRef.current && clampedBucketOffset >= bounds.maxOffset) {
      return
    }

    if (clampedBucketOffset >= bounds.maxOffset) {
      if (!isLooping) {
        onPlayingChange(false)
        return
      }

      const timeoutId = window.setTimeout(() => {
        onBucketOffsetChange(bounds.minOffset)
      }, playbackIntervalMs)

      return () => window.clearTimeout(timeoutId)
    }

    const timeoutId = window.setTimeout(() => {
      onBucketOffsetChange(clampedBucketOffset + 1)
    }, playbackIntervalMs)

    return () => window.clearTimeout(timeoutId)
  }, [
    bounds.maxOffset,
    bounds.minOffset,
    canPlay,
    clampedBucketOffset,
    isLooping,
    isPlaying,
    onBucketOffsetChange,
    onPlayingChange,
    playbackIntervalMs,
  ])

  const handleTogglePlayback = () => {
    if (!canPlay) return

    if (isPlaying) {
      onPlayingChange(false)
      pendingRestartRef.current = false
      return
    }

    if (clampedBucketOffset >= bounds.maxOffset) {
      pendingRestartRef.current = true
      onBucketOffsetChange(bounds.minOffset)
    }

    onPlayingChange(true)
  }
  const handlePlaybackIntervalInputChange = (
    event: React.ChangeEvent<HTMLInputElement>,
  ) => {
    const value = event.target.value
    setPlaybackIntervalInput(value)

    const nextInterval = Number(value)
    if (
      Number.isInteger(nextInterval) &&
      nextInterval > 0 &&
      nextInterval <= maxPlaybackIntervalMs
    ) {
      onPlaybackIntervalChange(nextInterval)
    }
  }

  return (
    <>
      <label className="sr-only" htmlFor={bucketSelectId}>
        Select {binLabel}
      </label>
      <div className="relative min-w-36">
        <select
          className="h-6 w-full appearance-none rounded border border-gray-500/15 bg-white/80 pl-2 pr-6 text-xs font-medium text-gray-800 shadow-xs outline-none hover:bg-white focus:border-blue-500 focus:ring-2 focus:ring-blue-500/20 dark:bg-gray-900/70 dark:text-gray-100 dark:hover:bg-gray-900"
          id={bucketSelectId}
          onChange={(event) =>
            onBucketOffsetChange(Number(event.currentTarget.value))
          }
          value={clampedBucketOffset}
        >
          {bucketOptions.map((option) => (
            <option key={option.offset} value={option.offset}>
              {option.label}
            </option>
          ))}
        </select>
        <CaretDownIcon className="pointer-events-none absolute right-1.5 top-1/2 size-3 -translate-y-1/2 text-gray-500" />
      </div>
      <div className="flex items-center overflow-hidden rounded border border-gray-500/15 bg-white/70 shadow-xs dark:bg-gray-900/70">
        <button
          aria-label={`First ${binLabel}`}
          className={navigatorButtonStyles}
          disabled={!canGoBack}
          onClick={() => onBucketOffsetChange(bounds.minOffset)}
          type="button"
        >
          <CaretDoubleLeftIcon className="size-3.5" />
        </button>
        <button
          aria-label={`Previous ${binLabel}`}
          className={borderedNavigatorButtonStyles}
          disabled={!canGoBack}
          onClick={() => onBucketOffsetChange(clampedBucketOffset - 1)}
          type="button"
        >
          <CaretLeftIcon className="size-3.5" />
        </button>
        <button
          aria-label={`Next ${binLabel}`}
          className={borderedNavigatorButtonStyles}
          disabled={!canGoForward}
          onClick={() => onBucketOffsetChange(clampedBucketOffset + 1)}
          type="button"
        >
          <CaretRightIcon className="size-3.5" />
        </button>
        <button
          aria-label={`Last ${binLabel}`}
          className={borderedNavigatorButtonStyles}
          disabled={!canGoForward}
          onClick={() => onBucketOffsetChange(bounds.maxOffset)}
          type="button"
        >
          <CaretDoubleRightIcon className="size-3.5" />
        </button>
      </div>
      <div className="flex max-w-[348px] flex-1 items-center overflow-hidden rounded border border-gray-500/15 bg-white/70 shadow-xs dark:bg-gray-900/70">
        <button
          aria-label={isPlaying ? 'Pause bucket playback' : 'Play buckets'}
          aria-pressed={isPlaying}
          className={navigatorButtonStyles}
          disabled={!canPlay}
          onClick={handleTogglePlayback}
          type="button"
        >
          {isPlaying ? (
            <PauseIcon className="size-3.5" />
          ) : (
            <PlayIcon className="size-3.5" />
          )}
        </button>
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <button
              aria-label="Bucket playback options"
              className={borderedNavigatorButtonStyles}
              type="button"
            >
              <DotsThreeIcon className="size-3.5" />
            </button>
          </DropdownMenuTrigger>
          <DropdownMenuContent
            align="start"
            className="z-50 min-w-[160px] rounded-md bg-white p-1.5 shadow-lg dark:bg-gray-800"
            collisionPadding={8}
            sideOffset={5}
          >
            <div className="mb-1 flex items-center justify-between px-0.5 text-xs font-medium">
              <span>Playback Speed</span>
            </div>
            {playbackSpeedOptions.map((option) => (
              <DropdownMenuItem
                className={menuItemStyles}
                key={option.intervalMs}
                onSelect={() => onPlaybackIntervalChange(option.intervalMs)}
              >
                <CheckIcon
                  className={
                    option.intervalMs === playbackIntervalMs
                      ? 'size-3 opacity-100'
                      : 'size-3 opacity-0'
                  }
                />
                <span>{option.label}</span>
              </DropdownMenuItem>
            ))}
            <div className="mt-1 px-0.5 pb-1">
              <label
                className="mb-1 block text-xs font-medium"
                htmlFor={playbackIntervalInputId}
              >
                Custom ms
              </label>
              <input
                className="h-6 w-full rounded border border-gray-500/20 bg-transparent px-1.5 text-xs outline-none focus:border-blue-500"
                id={playbackIntervalInputId}
                max={maxPlaybackIntervalMs}
                min={1}
                onChange={handlePlaybackIntervalInputChange}
                onKeyDown={(event) => event.stopPropagation()}
                step={1}
                type="number"
                value={playbackIntervalInput}
              />
            </div>
            <div className="my-1 h-px bg-gray-500/20" />
            <DropdownMenuItem
              className={menuItemStyles}
              onSelect={() => onLoopingChange(!isLooping)}
            >
              <CheckIcon
                className={
                  isLooping ? 'size-3 opacity-100' : 'size-3 opacity-0'
                }
              />
              <span>Loop</span>
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
        <div className="flex h-6 min-w-36 max-w-[300px] flex-1 items-center border-l border-gray-500/15 bg-gray-500/5 px-1.5">
          <input
            aria-label={`Scrub ${binLabel}s`}
            className="block h-6 w-full appearance-none bg-transparent outline-none
              [&::-moz-range-thumb]:h-4 [&::-moz-range-thumb]:w-4 [&::-moz-range-thumb]:rounded-full [&::-moz-range-thumb]:border-2 [&::-moz-range-thumb]:border-white [&::-moz-range-thumb]:bg-blue-600 [&::-moz-range-thumb]:shadow-md
              [&::-moz-range-track]:h-1.5 [&::-moz-range-track]:rounded-full [&::-moz-range-track]:bg-gray-500/15
              [&::-webkit-slider-runnable-track]:h-1.5 [&::-webkit-slider-runnable-track]:rounded-full [&::-webkit-slider-runnable-track]:bg-gray-500/15
              [&::-webkit-slider-thumb]:-mt-[5px] [&::-webkit-slider-thumb]:h-4 [&::-webkit-slider-thumb]:w-4 [&::-webkit-slider-thumb]:appearance-none [&::-webkit-slider-thumb]:rounded-full [&::-webkit-slider-thumb]:border-2 [&::-webkit-slider-thumb]:border-white [&::-webkit-slider-thumb]:bg-blue-600 [&::-webkit-slider-thumb]:shadow-md
              dark:[&::-moz-range-thumb]:border-gray-900 dark:[&::-moz-range-track]:bg-gray-100/15
              dark:[&::-webkit-slider-runnable-track]:bg-gray-100/15 dark:[&::-webkit-slider-thumb]:border-gray-900"
            list={bucketRangeListId}
            max={bounds.maxOffset}
            min={bounds.minOffset}
            onChange={(event) =>
              onBucketOffsetChange(Number(event.currentTarget.value))
            }
            step={1}
            type="range"
            value={clampedBucketOffset}
          />
        </div>
      </div>
      <datalist id={bucketRangeListId}>
        {bucketOptions.map((option) => (
          <option key={option.offset} value={option.offset} />
        ))}
      </datalist>
    </>
  )
}
````

## File: src/components/npm-stats/npmChartTooltip.ts
````typescript
import type { ChartPoint, ChartTooltipContentContext } from '@tanstack/charts'
import type { BarOrientation, ViewMode } from './shared'

export type NpmHistoryPoint = {
  color: string
  date: Date
  id: string
  label: string
  seriesName: string
  seriesLabel?: string
  value: number
}

export type NpmStackedHistoryPoint = NpmHistoryPoint & {
  fill: string
  value1: number
  value2: number
}

export type NpmBarPoint = {
  category: string
  color: string
  fill: string
  id: string
  label: string
  seriesName: string
  value1: number
  value2: number
}

type NpmChartDatum = NpmHistoryPoint | NpmStackedHistoryPoint | NpmBarPoint

export function getNpmChartTooltipContent(
  points: ReadonlyArray<ChartPoint<NpmChartDatum>>,
  context: ChartTooltipContentContext,
  input: {
    barOrientation: BarOrientation
    viewMode: ViewMode
    xLabel?: string
    yLabel?: string
    yFormat?: (value: number) => string
  },
) {
  const first = points[0]
  if (!first) return { rows: [] }

  const getLabel = (point: ChartPoint<NpmChartDatum>) =>
    ('seriesLabel' in point.datum ? point.datum.seriesLabel : undefined) ??
    point.groupLabel

  const formatValue = (point: ChartPoint<NpmChartDatum>) => {
    const datum = point.datum
    const value = 'value1' in datum ? datum.value2 - datum.value1 : datum.value

    return input.yFormat?.(value) ?? value.toLocaleString()
  }

  if (points.length > 1) {
    const title =
      input.viewMode === 'history'
        ? `Date: ${context.formatX(first.xValue)}`
        : input.barOrientation === 'horizontal'
          ? context.formatY(first.yValue)
          : context.formatX(first.xValue)

    return {
      title,
      rows: points.map((point) => ({
        active: point === context.primaryPoint,
        color: point.datum.color,
        label: getLabel(point),
        value: formatValue(point),
      })),
    }
  }

  return {
    title: getLabel(first),
    color: first.datum.color,
    rows:
      input.viewMode === 'history'
        ? [
            { label: 'Date', value: context.formatX(first.xValue) },
            {
              label: input.yLabel ?? 'Downloads',
              value: formatValue(first),
            },
          ]
        : [
            {
              label: input.xLabel ?? input.yLabel ?? 'Downloads',
              value: formatValue(first),
            },
          ],
  }
}
````

## File: src/components/npm-stats/npmQueryOptions.ts
````typescript
import { keepPreviousData, queryOptions } from '@tanstack/react-query'
import type {
  BinType,
  NpmQueryData,
  PackageGroup,
  TimeRange,
  ViewMode,
} from './shared'
import { fetchNpmDownloadsBulk } from '~/utils/stats-queries.functions'
import {
  formatNpmStatsDate,
  getHistoryStartDate,
  getLatestBucketWindow,
  getUtcDay,
  getUtcToday,
} from './binning'

/**
 * Shared TanStack Query options for fetching NPM download statistics.
 * Used by both the main /stats/npm page and library-specific NPM stats pages.
 */
export function npmQueryOptions({
  packageGroups,
  range,
}: {
  packageGroups: PackageGroup[]
  range: TimeRange
  viewMode?: ViewMode
  binType?: BinType
  bucketOffset?: number
}) {
  const now = getUtcToday()
  const dateWindow = {
    startDate: getHistoryStartDate(range, now),
    endDate: now,
  }

  return queryOptions({
    queryKey: [
      'npm-stats',
      packageGroups.map((pg) => ({
        packages: pg.packages.map((p) => ({ name: p.name })),
      })),
      range,
    ],
    queryFn: async (): Promise<NpmQueryData> => {
      try {
        // Make a single bulk request for all packages
        const results = await fetchNpmDownloadsBulk({
          data: {
            packageGroups: packageGroups.map((pg) => ({
              packages: pg.packages.map((p) => ({
                name: p.name,
                hidden: p.hidden,
              })),
            })),
            startDate: formatNpmStatsDate(dateWindow.startDate),
            endDate: formatNpmStatsDate(dateWindow.endDate),
          },
        })

        // Process results to match the expected format
        return results.map((result: any, groupIndex: number) => {
          let actualStartDate = dateWindow.startDate

          // Find the earliest non-zero download for this package group
          for (const pkg of result.packages) {
            const firstNonZero = pkg.downloads.find((d: any) => d.downloads > 0)
            if (firstNonZero) {
              const firstNonZeroDate = getUtcDay(new Date(firstNonZero.day))
              if (firstNonZeroDate < actualStartDate) {
                actualStartDate = firstNonZeroDate
              }
            }
          }

          return {
            packages: result.packages.map((pkg: any) => ({
              ...packageGroups[groupIndex]?.packages.find(
                (p) => p.name === pkg.name,
              ),
              downloads: pkg.downloads,
            })),
            start: formatNpmStatsDate(actualStartDate),
            end: formatNpmStatsDate(dateWindow.endDate),
            error: result.error ?? undefined,
            actualStartDate,
          }
        })
      } catch (error) {
        console.error('Failed to fetch npm stats:', error)
        // Return error state for all package groups
        return packageGroups.map((packageGroup) => ({
          packages: packageGroup.packages.map((pkg) => ({
            ...pkg,
            downloads: [],
          })),
          start: formatNpmStatsDate(dateWindow.startDate),
          end: formatNpmStatsDate(dateWindow.endDate),
          error: 'Failed to fetch package data (see console for details)',
          actualStartDate: dateWindow.startDate,
        }))
      }
    },
    placeholderData: keepPreviousData,
  })
}

export function selectLatestBucketQueryData({
  queryData,
  binType,
  bucketOffset,
  now = getUtcToday(),
}: {
  queryData: NpmQueryData | undefined
  binType: BinType
  bucketOffset: number
  now?: Date
}) {
  if (!queryData) return undefined

  const dateWindow = getLatestBucketWindow({ binType, bucketOffset, now })
  const startTime = dateWindow.startDate.getTime()
  const endTime = dateWindow.endDate.getTime()

  return queryData.map((packageGroup) => ({
    ...packageGroup,
    packages: packageGroup.packages.map((pkg) => ({
      ...pkg,
      downloads: pkg.downloads.filter((download) => {
        const dayTime = getUtcDay(new Date(download.day)).getTime()
        return dayTime >= startTime && dayTime <= endTime
      }),
    })),
    start: formatNpmStatsDate(dateWindow.startDate),
    end: formatNpmStatsDate(dateWindow.endDate),
    actualStartDate: dateWindow.startDate,
  }))
}
````

## File: src/components/npm-stats/NPMStatsChart.tsx
````typescript
import * as React from 'react'
import { createPortal } from 'react-dom'
import * as d3 from 'd3'
import {
  areaY,
  barX,
  barY,
  createChartScene,
  d3Curve,
  defineChart,
  lineY,
  ruleX,
  ruleY,
  type ChartLinearGradient,
} from '@tanstack/charts'
import { renderChartSvgWithResources } from '@tanstack/charts/svg/resources'
import { tooltip } from '@tanstack/charts/tooltip'
import { Chart } from '@tanstack/charts/react'
import { GIFEncoder, applyPalette, quantize } from 'gifenc'
import { getSvgDimensions } from './svgDimensions'
import {
  CheckIcon,
  CodeIcon,
  CopyIcon,
  DownloadIcon,
  ListIcon,
} from '@phosphor-icons/react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@radix-ui/react-dropdown-menu'
import { twMerge } from 'tailwind-merge'
import { Tooltip } from '~/components/Tooltip'
import { copyTextToClipboard } from '~/utils/browser-effects'
import {
  binningOptionsByType,
  getHistoryStartDate,
  getUtcDay,
  getUtcToday,
  NPM_STATS_START_DATE,
} from './binning'
import { selectLatestBucketQueryData } from './npmQueryOptions'
import type {
  BarOrientation,
  BinType,
  ChartType,
  LatestBarSort,
  NpmQueryData,
  PackageGroup,
  ShowDataMode,
  TimeRange,
  TransformMode,
  ViewMode,
} from './shared'
import {
  formatNumber,
  getBaselineDisplayName,
  getPackageColor,
  getPackageGroupLabel,
  hasPackageGroupLabel,
  isPackageGroupHidden,
} from './shared'
import { BASELINE_LINE_COLOR } from './BaselineSection'

import {
  getNpmChartTooltipContent,
  type NpmHistoryPoint,
  type NpmStackedHistoryPoint,
  type NpmBarPoint,
} from './npmChartTooltip'

const BASELINE_LINE_SERIES = '__baseline__'
const percentFormatter = new Intl.NumberFormat('en-US', {
  style: 'percent',
  maximumFractionDigits: 1,
})
const compactAxisNumberFormatter = d3.format('.3~s')
const chartUpdateTransitionDurationMs = 500

function formatCompactAxisNumber(value: number) {
  if (!Number.isFinite(value) || value === 0) return '0'
  return compactAxisNumberFormatter(value)
}

type StackedDateRow<Datum> = {
  date: Date
  pointsBySeries: Map<string, Datum>
  [seriesName: string]: Date | Map<string, Datum> | number
}

type TimelineRangeValue = {
  end: number | undefined
  start: number | undefined
}
type TimelineRangeScrubberProps = {
  dates: Array<Date>
  endIndex: number
  marginLeft: number
  marginRight: number
  onRangeChange: (startIndex: number, endIndex: number) => void
  startIndex: number
}
type TimelineScrubberDragState = {
  anchorIndex: number
  focusIndex: number
}
type TimelineScrubberPanState = {
  anchorIndex: number
  endIndex: number
  startIndex: number
}

const timelineScrubberDateFormatter = new Intl.DateTimeFormat('en-US', {
  day: 'numeric',
  month: 'short',
  year: 'numeric',
})
const timelineScrubberInputStyles =
  'pointer-events-none absolute inset-0 h-8 w-full appearance-none bg-transparent outline-none [&::-moz-range-thumb]:pointer-events-auto [&::-moz-range-thumb]:h-3 [&::-moz-range-thumb]:w-3 [&::-moz-range-thumb]:cursor-ew-resize [&::-moz-range-thumb]:border [&::-moz-range-thumb]:border-gray-400 [&::-moz-range-thumb]:bg-gray-300 [&::-moz-range-thumb]:shadow-xs [&::-moz-range-track]:h-3 [&::-moz-range-track]:bg-transparent [&::-webkit-slider-runnable-track]:h-3 [&::-webkit-slider-runnable-track]:bg-transparent [&::-webkit-slider-thumb]:pointer-events-auto [&::-webkit-slider-thumb]:mt-0 [&::-webkit-slider-thumb]:h-3 [&::-webkit-slider-thumb]:w-3 [&::-webkit-slider-thumb]:cursor-ew-resize [&::-webkit-slider-thumb]:appearance-none [&::-webkit-slider-thumb]:border [&::-webkit-slider-thumb]:border-gray-400 [&::-webkit-slider-thumb]:bg-gray-300 [&::-webkit-slider-thumb]:shadow-xs dark:[&::-moz-range-thumb]:border-gray-500 dark:[&::-moz-range-thumb]:bg-gray-700 dark:[&::-webkit-slider-thumb]:border-gray-500 dark:[&::-webkit-slider-thumb]:bg-gray-700'
const timelineScrubberStartInputStyles =
  '[&::-moz-range-thumb]:rounded-l-sm [&::-moz-range-thumb]:rounded-r-[1px] [&::-webkit-slider-thumb]:rounded-l-sm [&::-webkit-slider-thumb]:rounded-r-[1px]'
const timelineScrubberEndInputStyles =
  '[&::-moz-range-thumb]:rounded-l-[1px] [&::-moz-range-thumb]:rounded-r-sm [&::-webkit-slider-thumb]:rounded-l-[1px] [&::-webkit-slider-thumb]:rounded-r-sm'

function clampTimelineIndex(index: number, maxIndex: number) {
  return Math.max(0, Math.min(index, maxIndex))
}

function getTimelineIndexForTime({
  dates,
  fallbackIndex,
  time,
}: {
  dates: Array<Date>
  fallbackIndex: number
  time: number | undefined
}) {
  if (time === undefined || !Number.isFinite(time) || dates.length === 0) {
    return fallbackIndex
  }

  let low = 0
  let high = dates.length - 1

  while (low <= high) {
    const middle = Math.floor((low + high) / 2)
    const middleTime = dates[middle]?.getTime()

    if (middleTime === undefined) break
    if (middleTime === time) return middle
    if (middleTime < time) {
      low = middle + 1
    } else {
      high = middle - 1
    }
  }

  const nextIndex = clampTimelineIndex(low, dates.length - 1)
  const previousIndex = clampTimelineIndex(low - 1, dates.length - 1)
  const nextTime = dates[nextIndex]?.getTime()
  const previousTime = dates[previousIndex]?.getTime()
  const nextDistance =
    nextTime === undefined
      ? Number.POSITIVE_INFINITY
      : Math.abs(nextTime - time)
  const previousDistance =
    previousTime === undefined
      ? Number.POSITIVE_INFINITY
      : Math.abs(previousTime - time)

  return previousDistance <= nextDistance ? previousIndex : nextIndex
}

function formatTimelineScrubberDate(date: Date | undefined) {
  return date ? timelineScrubberDateFormatter.format(date) : ''
}

function getTimelineIndexFromPointer({
  clientX,
  element,
  maxIndex,
}: {
  clientX: number
  element: HTMLElement
  maxIndex: number
}) {
  const box = element.getBoundingClientRect()
  const progress = box.width > 0 ? (clientX - box.left) / box.width : 0

  return clampTimelineIndex(Math.round(progress * maxIndex), maxIndex)
}

function getInsideOutOrderByLatestValue({
  latestValues,
  sums,
}: {
  latestValues: Array<number>
  sums: Array<number>
}) {
  const order = d3
    .range(latestValues.length)
    .sort(
      (a, b) =>
        d3.descending(latestValues[a] ?? 0, latestValues[b] ?? 0) ||
        d3.ascending(a, b),
    )
  const tops: Array<number> = []
  const bottoms: Array<number> = []
  let topSum = 0
  let bottomSum = 0

  order.forEach((index) => {
    const sum = sums[index] ?? 0

    if (topSum < bottomSum) {
      topSum += sum
      tops.push(index)
      return
    }

    bottomSum += sum
    bottoms.push(index)
  })

  return bottoms.reverse().concat(tops)
}

function getSubPackageColorForPackages({
  groupName,
  packageIndex,
  packages,
}: {
  groupName: string
  packageIndex: number
  packages: Array<PackageGroup>
}) {
  const groupColor = getPackageColor(groupName, packages)
  const color = d3.color(groupColor)
  if (!color || packageIndex === 0) return groupColor

  return packageIndex % 2 === 0
    ? color.darker(packageIndex * 0.25).toString()
    : color.brighter(packageIndex * 0.35).toString()
}

type ChartExportFormat = 'svg' | 'png' | 'jpeg' | 'gif' | 'webm'
type ColorLegendEntry = {
  label: string
  color: string
}
type ExportLegendItem = {
  color: string
  label: string
}
type ExportLegendLayoutItem = {
  item: ExportLegendItem
  width: number
}
type ExportLegendLayoutRow = {
  items: Array<ExportLegendLayoutItem>
  width: number
}
type ChartFillGradient = {
  bottomOpacity: number
  color: string
  id: string
  topOpacity: number
}
type ExportBackgroundMode = 'opaque' | 'transparent'
type ExportStyle = {
  backgroundColor: string
  backgroundMode: ExportBackgroundMode
  color: string
}
type SerializedExportSvg = {
  height: number
  source: string
  width: number
}
type SerializedAnimationFrame = SerializedExportSvg & {
  delayMs: number
}
type LatestBucketOffsetBounds = {
  maxOffset: number
  minOffset: number
}
export type NpmStatsChartEmbedOptions = {
  includeTimelineRange: boolean
  lockWidth: boolean
  showLegend: boolean
}
export type NpmStatsChartEmbedConfig = {
  buildUrl: (options: NpmStatsChartEmbedOptions) => string
  hasTimelineRange: boolean
  hasWidth: boolean
}

const chartExportOptions = [
  { value: 'svg', label: 'SVG' },
  { value: 'png', label: 'PNG' },
  { value: 'jpeg', label: 'JPEG' },
  { value: 'gif', label: 'GIF' },
  { value: 'webm', label: 'WebM' },
] as const satisfies ReadonlyArray<{
  value: ChartExportFormat
  label: string
}>
const animatedExportMaxFrames = 240
const animatedExportMinDelayMs = 35
const animatedExportMaxDelayMs = 100
const axisTickAverageCharacterWidth = 6.4
const axisTickLineHeight = 12
const chartActionButtonStyles =
  'flex size-6 items-center justify-center rounded bg-gray-500/10 text-gray-600 hover:bg-gray-500/20 hover:text-blue-500 dark:text-gray-400 dark:hover:text-gray-100'
const chartActionDropdownContentStyles =
  'z-50 min-w-[120px] rounded-md bg-white p-1.5 shadow-lg dark:bg-gray-800'
const chartActionDropdownItemStyles =
  'flex w-full cursor-pointer items-center rounded px-1.5 py-1 text-left text-xs outline-none hover:bg-gray-500/20 data-highlighted:bg-gray-500/20 data-highlighted:text-blue-500'
const chartEmbedInputStyles =
  'w-full rounded border border-gray-500/20 bg-gray-50 px-2 py-1.5 font-mono text-[11px] leading-snug outline-none focus:border-blue-500 dark:bg-gray-900'

function clampNumber(value: number, min: number, max: number) {
  return Math.max(min, Math.min(value, max))
}

function getEstimatedAxisLabelWidth(label: string) {
  return Math.ceil(label.length * axisTickAverageCharacterWidth)
}

function getCategoricalAxisLeftMargin({
  labels,
  width,
}: {
  labels: Array<string>
  width: number
}) {
  const longestLabelWidth =
    d3.max(labels, (label) => getEstimatedAxisLabelWidth(label)) ?? 0
  const maxMargin = clampNumber(Math.round(width * 0.38), 110, 280)

  return clampNumber(longestLabelWidth + 26, 90, maxMargin)
}

function getVerticalCategoricalXAxisLayout({
  height,
  labels,
  marginLeft,
  marginRight,
  width,
}: {
  height: number
  labels: Array<string>
  marginLeft: number
  marginRight: number
  width: number
}) {
  const availableWidth = Math.max(0, width - marginLeft - marginRight)
  const tickSlotWidth = labels.length ? availableWidth / labels.length : 0
  const longestLabelWidth =
    d3.max(labels, (label) => getEstimatedAxisLabelWidth(label)) ?? 0
  const shouldRotateTicks =
    labels.length > 0 && longestLabelWidth > Math.max(36, tickSlotWidth - 8)

  if (!shouldRotateTicks) {
    return {
      marginBottom: 64,
      tickRotate: undefined,
    }
  }

  const tickAngle = Math.PI / 6
  const rotatedTickHeight = Math.ceil(
    longestLabelWidth * Math.sin(tickAngle) +
      axisTickLineHeight * Math.cos(tickAngle),
  )
  const maxMarginBottom = clampNumber(Math.round(height * 0.45), 88, 160)

  return {
    marginBottom: clampNumber(44 + rotatedTickHeight, 76, maxMarginBottom),
    tickRotate: -30,
  }
}

function getChartContentHeight({
  reservedHeight,
  totalHeight,
}: {
  reservedHeight: number
  totalHeight: number | undefined
}) {
  return Math.max(1, Math.floor((totalHeight ?? 0) - reservedHeight))
}

function getExportFileName(format: ChartExportFormat) {
  if (format === 'jpeg') return 'npm-stats-chart.jpg'
  return `npm-stats-chart.${format}`
}

function isAnimatedExportFormat(
  format: ChartExportFormat,
): format is 'gif' | 'webm' {
  return format === 'gif' || format === 'webm'
}

function getStableIdPart(value: string) {
  let hash = 2166136261

  for (let index = 0; index < value.length; index++) {
    hash = Math.imul(hash ^ value.charCodeAt(index), 16777619)
  }

  return (hash >>> 0).toString(36)
}

function getChartFillGradients({
  bottomOpacity = 0.1,
  entries,
  idPrefix,
  topOpacity = 0.8,
}: {
  bottomOpacity?: number
  entries: Array<ColorLegendEntry>
  idPrefix: string
  topOpacity?: number
}) {
  const seenColors = new Set<string>()
  const gradients: Array<ChartFillGradient> = []

  entries.forEach((entry) => {
    if (seenColors.has(entry.color)) return

    seenColors.add(entry.color)
    gradients.push({
      bottomOpacity,
      color: entry.color,
      id: `${idPrefix}-${getStableIdPart(entry.color)}`,
      topOpacity,
    })
  })

  return gradients
}

function getTanStackGradients(
  gradients: Array<ChartFillGradient>,
): Array<ChartLinearGradient> {
  return gradients.map((gradient) => ({
    id: gradient.id,
    stops: [
      {
        offset: 0,
        color: gradient.color,
        opacity: gradient.bottomOpacity,
      },
      {
        offset: 1,
        color: gradient.color,
        opacity: gradient.topOpacity,
      },
    ],
  }))
}

function getChartFillGradientUrl({
  color,
  gradients,
}: {
  color: string
  gradients: Array<ChartFillGradient>
}) {
  const gradient = gradients.find((entry) => entry.color === color)

  return gradient ? `url(#${gradient.id})` : color
}

function getUniqueColorLegendEntries(entries: Array<ColorLegendEntry>) {
  const colorByLabel = new Map<string, string>()

  entries.forEach((entry) => {
    if (!colorByLabel.has(entry.label)) {
      colorByLabel.set(entry.label, entry.color)
    }
  })

  return [...colorByLabel].map(([label, color]) => ({ label, color }))
}

function getChartSvg(svg: SVGSVGElement | null) {
  return svg ?? undefined
}

function getLatestExportPackageData({
  binType,
  packages,
  queryData,
}: {
  binType: BinType
  packages: Array<PackageGroup>
  queryData: NpmQueryData
}) {
  const binUnit = binningOptionsByType[binType].bin
  const earliestQueryStart = queryData
    .map((pkg) => getUtcDay(new Date(pkg.start)))
    .sort((a, b) => a.getTime() - b.getTime())[0]
  const startDate = binUnit.floor(earliestQueryStart || NPM_STATS_START_DATE)

  const combinedPackageGroups = queryData.map((queryPackageGroup, index) => {
    const packageGroupWithHidden = packages[index]
    const shouldAlwaysIncludeFirstPackage =
      !packageGroupWithHidden ||
      !hasPackageGroupLabel(packageGroupWithHidden) ||
      !!packageGroupWithHidden.baseline
    const visiblePackages = queryPackageGroup.packages.filter((p, i) => {
      const hiddenState = packageGroupWithHidden?.packages.find(
        (pg) => pg.name === p.name,
      )?.hidden
      return (i === 0 && shouldAlwaysIncludeFirstPackage) || !hiddenState
    })
    const groupName = packageGroupWithHidden
      ? getPackageGroupLabel(packageGroupWithHidden)
      : queryPackageGroup.packages[0]?.name || 'Unknown'
    const downloadsByDate = new Map<number, number>()

    visiblePackages.forEach((pkg) => {
      pkg.downloads.forEach((d) => {
        const date = getUtcDay(new Date(d.day))
        if (date < startDate) return

        downloadsByDate.set(
          date.getTime(),
          (downloadsByDate.get(date.getTime()) || 0) + d.downloads,
        )
      })
    })

    const packageDownloads = visiblePackages.map((pkg) => {
      const packageName = pkg.name || groupName
      const packageDownloadsByDate = new Map<number, number>()

      pkg.downloads.forEach((d) => {
        const date = getUtcDay(new Date(d.day))
        if (date < startDate) return

        packageDownloadsByDate.set(
          date.getTime(),
          (packageDownloadsByDate.get(date.getTime()) || 0) + d.downloads,
        )
      })

      return {
        packageName,
        downloads: Array.from(packageDownloadsByDate.entries()).map(
          ([date, downloads]) => ({
            date: getUtcDay(new Date(date)),
            downloads,
          }),
        ),
      }
    })

    return {
      groupName,
      packageDownloads,
      downloads: Array.from(downloadsByDate.entries()).map(
        ([date, downloads]) => [getUtcDay(new Date(date)), downloads],
      ) as Array<[Date, number]>,
    }
  })

  return combinedPackageGroups
    .map((packageGroup) => {
      const binned = d3.sort(
        d3.rollup(
          packageGroup.downloads,
          (v) => d3.sum(v, (d) => d[1]),
          (d) => binUnit.floor(d[0]),
        ),
        (d) => d[0],
      )

      return {
        ...packageGroup,
        downloads: binned.map((d) => ({
          name: packageGroup.groupName,
          date: getUtcDay(new Date(d[0])),
          downloads: d[1],
        })),
        packageDownloads: packageGroup.packageDownloads.map((pkg) => {
          const packageBinned = d3.sort(
            d3.rollup(
              pkg.downloads,
              (v) => d3.sum(v, (d) => d.downloads),
              (d) => binUnit.floor(d.date),
            ),
            (d) => d[0],
          )

          return {
            ...pkg,
            groupName: packageGroup.groupName,
            downloads: packageBinned.map((d) => ({
              groupName: packageGroup.groupName,
              packageName: pkg.packageName,
              date: getUtcDay(new Date(d[0])),
              downloads: d[1],
            })),
          }
        }),
      }
    })
    .filter((_, index) => {
      const packageGroupWithHidden = packages[index]
      return packageGroupWithHidden
        ? !isPackageGroupHidden(packageGroupWithHidden)
        : true
    })
}

function createLatestBarExportChart({
  barOrientation,
  barSort,
  binType,
  chartType,
  gradientIdPrefix,
  height,
  ownerDocument,
  packages,
  queryData,
  width,
}: {
  barOrientation: BarOrientation
  barSort: LatestBarSort
  binType: BinType
  chartType: ChartType
  gradientIdPrefix: string
  height: number
  ownerDocument: Document
  packages: Array<PackageGroup>
  queryData: NpmQueryData
  width: number
}) {
  const filteredPackageData = getLatestExportPackageData({
    binType,
    packages,
    queryData,
  })
  const latestBarGroups = filteredPackageData
    .map((packageGroup) => ({
      packageGroup,
      name: packageGroup.groupName,
      downloads: d3.sum(packageGroup.downloads, (d) => d.downloads),
    }))
    .filter((group) => group.downloads > 0)
    .sort((a, b) =>
      barSort === 'value'
        ? d3.descending(a.downloads, b.downloads) ||
          a.name.localeCompare(b.name)
        : a.name.localeCompare(b.name),
    )
  const latestBarDomain = latestBarGroups.map((group) => group.name)
  const latestGroupedBarData = latestBarGroups.map((group) => ({
    name: group.name,
    seriesName: group.name,
    downloads: group.downloads,
    label: `${group.name}: ${formatNumber(group.downloads)}`,
  }))
  const latestStackedBarData = latestBarGroups.flatMap((group) => {
    let y0 = 0
    const sortedPackageDownloads = group.packageGroup.packageDownloads
      .map((pkg, packageIndex) => ({
        ...pkg,
        packageIndex,
        totalDownloads: d3.sum(pkg.downloads, (d) => d.downloads),
      }))
      .filter((pkg) => pkg.totalDownloads > 0)
      .sort((a, b) => b.totalDownloads - a.totalDownloads)

    return sortedPackageDownloads.map((pkg) => {
      const point = {
        groupName: group.name,
        packageName: pkg.packageName,
        packageIndex: pkg.packageIndex,
        downloads: pkg.totalDownloads,
        y0,
        y1: y0 + pkg.totalDownloads,
        label: `${pkg.packageName}: ${formatNumber(pkg.totalDownloads)}`,
      }
      y0 += pkg.totalDownloads
      return point
    })
  })
  const isStackedBar = chartType === 'stacked-bar'
  const isHorizontalBar = barOrientation === 'horizontal'
  const marginRight = 10
  const marginLeft = isHorizontalBar
    ? getCategoricalAxisLeftMargin({ labels: latestBarDomain, width })
    : 70
  const verticalXAxisLayout = getVerticalCategoricalXAxisLayout({
    height,
    labels: latestBarDomain,
    marginLeft,
    marginRight,
    width,
  })
  const getSeriesColor = (d: { name?: string }) =>
    d.name ? getPackageColor(d.name, packages) : 'currentColor'
  const getSubPackageColor = (d: { groupName: string; packageIndex: number }) =>
    getSubPackageColorForPackages({ ...d, packages })
  const colorLegendEntries = getUniqueColorLegendEntries(
    isStackedBar
      ? latestStackedBarData.map((bar) => ({
          label: `${bar.groupName} / ${bar.packageName}`,
          color: getSubPackageColor(bar),
        }))
      : latestGroupedBarData.map((bar) => ({
          label: bar.name,
          color: getPackageColor(bar.name, packages),
        })),
  )
  const fillGradients = getChartFillGradients({
    bottomOpacity: 0.7,
    entries: colorLegendEntries,
    idPrefix: gradientIdPrefix,
    topOpacity: 1,
  })
  const getSeriesGradientFill = (d: { name?: string }) =>
    getChartFillGradientUrl({
      color: getSeriesColor(d),
      gradients: fillGradients,
    })
  const getSubPackageGradientFill = (d: {
    groupName: string
    packageIndex: number
  }) =>
    getChartFillGradientUrl({
      color: getSubPackageColor(d),
      gradients: fillGradients,
    })
  const maxDownloads =
    (isStackedBar
      ? d3.max(latestStackedBarData, (datum) => datum.y1)
      : d3.max(latestGroupedBarData, (datum) => datum.downloads)) ?? 1
  const categoricalScale = d3
    .scaleBand<string>()
    .domain(latestBarDomain)
    .paddingInner(0.1)
    .paddingOuter(0.05)
  const numericScale = d3.scaleLinear().domain([0, maxDownloads]).nice(7)
  const chart = defineChart({
    marks: isHorizontalBar
      ? [
          ruleX([0], {
            stroke: 'currentColor',
            strokeWidth: 1.5,
            strokeOpacity: 0.5,
          }),
          isStackedBar
            ? barX(latestStackedBarData, {
                id: 'latest-stacked-bars-x',
                y: 'groupName',
                x1: 'y0',
                x2: 'y1',
                key: (datum) =>
                  `${datum.groupName}:${datum.packageName}:${datum.packageIndex}`,
                fill: getSubPackageGradientFill,
                inset: 1,
              })
            : barX(latestGroupedBarData, {
                id: 'latest-bars-x',
                x: 'downloads',
                y: 'name',
                key: 'name',
                fill: getSeriesGradientFill,
                inset: 2,
                radius: 2,
              }),
        ]
      : [
          ruleY([0], {
            stroke: 'currentColor',
            strokeWidth: 1.5,
            strokeOpacity: 0.5,
          }),
          isStackedBar
            ? barY(latestStackedBarData, {
                id: 'latest-stacked-bars-y',
                x: 'groupName',
                y1: 'y0',
                y2: 'y1',
                key: (datum) =>
                  `${datum.groupName}:${datum.packageName}:${datum.packageIndex}`,
                fill: getSubPackageGradientFill,
                inset: 1,
              })
            : barY(latestGroupedBarData, {
                id: 'latest-bars-y',
                x: 'name',
                y: 'downloads',
                key: 'name',
                fill: getSeriesGradientFill,
                inset: 2,
                radius: 2,
              }),
        ],
    scales: {
      x: isHorizontalBar
        ? {
            scale: numericScale,
            axis: {
              label: { text: 'Downloads', offset: 35 },
              ticks: { format: formatCompactAxisNumber },
            },
            grid: true,
          }
        : {
            scale: categoricalScale,
            axis: {
              tickLabels: { rotate: verticalXAxisLayout.tickRotate },
            },
          },
      y: isHorizontalBar
        ? { scale: categoricalScale }
        : {
            scale: numericScale,
            axis: {
              label: 'Downloads',
              ticks: { format: formatCompactAxisNumber },
            },
            grid: true,
          },
    },
    gradients: getTanStackGradients(fillGradients),
    margin: {
      top: 20,
      right: marginRight,
      bottom: isHorizontalBar ? 64 : verticalXAxisLayout.marginBottom,
      left: marginLeft,
    },
  })
  const source = renderChartSvgWithResources(
    createChartScene(chart, { width, height }),
    {
      ariaLabel: 'npm download totals',
      idPrefix: gradientIdPrefix,
    },
  )
  const container = ownerDocument.createElement('div')
  container.innerHTML = source
  const svg = container.querySelector<SVGSVGElement>('svg')
  return svg ?? undefined
}

function isTransparentColor(color: string) {
  const normalizedColor = color.trim().toLowerCase()

  return (
    normalizedColor === '' ||
    normalizedColor === 'transparent' ||
    normalizedColor === 'rgba(0, 0, 0, 0)' ||
    normalizedColor === 'rgba(0,0,0,0)' ||
    /^rgba\(.+,\s*0(?:\.0+)?\)$/.test(normalizedColor) ||
    /^rgb\(.+\/\s*0(?:%|\.0+)?\)$/.test(normalizedColor)
  )
}

function getEffectiveBackgroundColor(element: Element) {
  const defaultView = element.ownerDocument.defaultView
  if (!defaultView) return '#ffffff'

  let current: Element | null = element
  while (current) {
    const backgroundColor =
      defaultView.getComputedStyle(current).backgroundColor
    if (!isTransparentColor(backgroundColor)) {
      return backgroundColor
    }

    current = current.parentElement
  }

  const bodyBackgroundColor = defaultView.getComputedStyle(
    element.ownerDocument.body,
  ).backgroundColor
  if (!isTransparentColor(bodyBackgroundColor)) {
    return bodyBackgroundColor
  }

  return element.ownerDocument.documentElement.classList.contains('dark')
    ? '#030712'
    : '#ffffff'
}

function getExportStyle({
  format,
  svg,
}: {
  format: ChartExportFormat
  svg: SVGSVGElement
}): ExportStyle {
  const computedStyle = svg.ownerDocument.defaultView?.getComputedStyle(svg)
  const color = computedStyle?.color || 'currentColor'

  return {
    backgroundColor: getEffectiveBackgroundColor(svg),
    backgroundMode:
      format === 'jpeg' || isAnimatedExportFormat(format)
        ? 'opaque'
        : 'transparent',
    color,
  }
}

function getExportLegendItems({
  container,
  showLegend,
}: {
  container: HTMLDivElement | null
  showLegend: boolean
}) {
  if (!container || !showLegend) return []

  return [
    ...container.querySelectorAll<HTMLElement>('[data-chart-legend-item]'),
  ]
    .map((item): ExportLegendItem | undefined => {
      const label = item.textContent?.trim()
      const color = item.dataset.color
      if (!label || !color || color === 'none') return undefined

      return { color, label }
    })
    .filter((item): item is ExportLegendItem => item !== undefined)
}

function getEstimatedLegendLabelWidth(label: string) {
  return Math.ceil(label.length * 5.8)
}

function getLegendLayout({
  items,
  width,
}: {
  items: Array<ExportLegendItem>
  width: number
}) {
  const maxRowWidth = Math.max(1, width - 24)
  const itemGap = 12
  const rows: Array<ExportLegendLayoutRow> = []
  let currentRow: ExportLegendLayoutRow = { items: [], width: 0 }

  items.forEach((item) => {
    const itemWidth = 10 + 5 + getEstimatedLegendLabelWidth(item.label)
    const nextWidth = currentRow.items.length
      ? currentRow.width + itemGap + itemWidth
      : itemWidth

    if (currentRow.items.length && nextWidth > maxRowWidth) {
      rows.push(currentRow)
      currentRow = { items: [{ item, width: itemWidth }], width: itemWidth }
      return
    }

    currentRow.items.push({ item, width: itemWidth })
    currentRow.width = nextWidth
  })

  if (currentRow.items.length) {
    rows.push(currentRow)
  }

  return {
    height: rows.length ? 12 + rows.length * 14 + (rows.length - 1) * 5 + 8 : 0,
    itemGap,
    maxRowWidth,
    rows,
  }
}

function appendExportLegend({
  exportSvg,
  exportStyle,
  items,
  plotHeight,
  width,
}: {
  exportSvg: SVGSVGElement
  exportStyle: ExportStyle
  items: Array<ExportLegendItem>
  plotHeight: number
  width: number
}) {
  if (!items.length) return 0

  const ownerDocument = exportSvg.ownerDocument
  const layout = getLegendLayout({ items, width })
  const group = ownerDocument.createElementNS('http://www.w3.org/2000/svg', 'g')
  group.setAttribute('font-family', 'system-ui, sans-serif')
  group.setAttribute('font-size', '10')
  group.setAttribute('fill', exportStyle.color)

  let y = plotHeight + 12

  layout.rows.forEach((row) => {
    let x = 12 + Math.max(0, (layout.maxRowWidth - row.width) / 2)

    row.items.forEach(({ item, width: itemWidth }, index) => {
      if (index > 0) {
        x += layout.itemGap
      }

      const rect = ownerDocument.createElementNS(
        'http://www.w3.org/2000/svg',
        'rect',
      )
      rect.setAttribute('x', `${x}`)
      rect.setAttribute('y', `${y + 2}`)
      rect.setAttribute('width', '10')
      rect.setAttribute('height', '10')
      rect.setAttribute('fill', item.color)
      group.append(rect)

      const text = ownerDocument.createElementNS(
        'http://www.w3.org/2000/svg',
        'text',
      )
      text.setAttribute('x', `${x + 15}`)
      text.setAttribute('y', `${y + 7}`)
      text.setAttribute('dominant-baseline', 'central')
      text.textContent = item.label
      group.append(text)

      x += itemWidth
    })

    y += 19
  })

  exportSvg.append(group)
  return layout.height
}

function serializeSvg({
  exportStyle,
  legendItems,
  svg,
}: {
  exportStyle: ExportStyle
  legendItems: Array<ExportLegendItem>
  svg: SVGSVGElement
}): SerializedExportSvg | undefined {
  const clone = svg.cloneNode(true)
  if (!(clone instanceof SVGSVGElement)) return undefined

  const { height, width } = getSvgDimensions(svg)
  const ownerDocument = svg.ownerDocument
  const exportSvg = ownerDocument.createElementNS(
    'http://www.w3.org/2000/svg',
    'svg',
  )
  const legendHeight = legendItems.length
    ? getLegendLayout({ items: legendItems, width }).height
    : 0
  const exportHeight = height + legendHeight
  const background = ownerDocument.createElementNS(
    'http://www.w3.org/2000/svg',
    'rect',
  )

  exportSvg.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
  exportSvg.setAttribute('width', `${width}`)
  exportSvg.setAttribute('height', `${exportHeight}`)
  exportSvg.setAttribute('viewBox', `0 0 ${width} ${exportHeight}`)
  exportSvg.style.backgroundColor =
    exportStyle.backgroundMode === 'opaque'
      ? exportStyle.backgroundColor
      : 'transparent'
  exportSvg.style.color = exportStyle.color

  if (exportStyle.backgroundMode === 'opaque') {
    background.setAttribute('width', '100%')
    background.setAttribute('height', '100%')
    background.setAttribute('fill', exportStyle.backgroundColor)
    exportSvg.append(background)
  }

  clone.setAttribute('x', '0')
  clone.setAttribute('y', '0')
  clone.setAttribute('width', `${width}`)
  clone.setAttribute('height', `${height}`)
  clone.style.backgroundColor = 'transparent'
  clone.style.color = exportStyle.color
  exportSvg.append(clone)

  appendExportLegend({
    exportSvg,
    exportStyle,
    items: legendItems,
    plotHeight: height,
    width,
  })

  return {
    height: exportHeight,
    source: new XMLSerializer().serializeToString(exportSvg),
    width,
  }
}

function downloadBlob({
  blob,
  fileName,
  ownerDocument,
}: {
  blob: Blob
  fileName: string
  ownerDocument: Document
}) {
  const url = URL.createObjectURL(blob)
  const anchor = ownerDocument.createElement('a')
  anchor.href = url
  anchor.download = fileName
  anchor.style.display = 'none'
  ownerDocument.body.append(anchor)
  anchor.click()
  anchor.remove()

  const defaultView = ownerDocument.defaultView
  if (defaultView) {
    defaultView.setTimeout(() => URL.revokeObjectURL(url), 0)
  } else {
    URL.revokeObjectURL(url)
  }
}

function canvasToBlob({
  canvas,
  type,
}: {
  canvas: HTMLCanvasElement
  type: 'image/png' | 'image/jpeg'
}) {
  return new Promise<Blob | undefined>((resolve) => {
    canvas.toBlob(
      (blob) => {
        resolve(blob ?? undefined)
      },
      type,
      0.92,
    )
  })
}

function waitForMs(defaultView: Window, durationMs: number) {
  return new Promise<void>((resolve) => {
    defaultView.setTimeout(resolve, durationMs)
  })
}

function getAnimatedExportFrameDelayMs(playbackIntervalMs: number | undefined) {
  const interval = playbackIntervalMs ?? animatedExportMaxDelayMs

  return Math.max(
    animatedExportMinDelayMs,
    Math.min(interval, animatedExportMaxDelayMs),
  )
}

function getAnimatedExportBucketOffsets(bounds: LatestBucketOffsetBounds) {
  const bucketCount = bounds.maxOffset - bounds.minOffset + 1
  if (bucketCount <= 1) return [bounds.maxOffset]
  if (bucketCount <= animatedExportMaxFrames) {
    return d3.range(bounds.minOffset, bounds.maxOffset + 1)
  }

  const offsets: Array<number> = []
  const seenOffsets = new Set<number>()

  for (let index = 0; index < animatedExportMaxFrames; index++) {
    const progress = index / (animatedExportMaxFrames - 1)
    const offset = Math.round(
      bounds.minOffset + progress * (bounds.maxOffset - bounds.minOffset),
    )

    if (!seenOffsets.has(offset)) {
      seenOffsets.add(offset)
      offsets.push(offset)
    }
  }

  return offsets
}

async function drawSerializedSvgToCanvas({
  canvas,
  exportStyle,
  ownerDocument,
  serializedSvg,
}: {
  canvas: HTMLCanvasElement
  exportStyle: ExportStyle
  ownerDocument: Document
  serializedSvg: SerializedExportSvg
}) {
  const defaultView = ownerDocument.defaultView
  const ImageConstructor = defaultView?.Image
  if (!ImageConstructor) return undefined

  if (
    canvas.width !== serializedSvg.width ||
    canvas.height !== serializedSvg.height
  ) {
    canvas.width = serializedSvg.width
    canvas.height = serializedSvg.height
  }

  const context = canvas.getContext('2d')
  if (!context) return undefined

  const image = new ImageConstructor()
  const sourceBlob = new Blob([serializedSvg.source], {
    type: 'image/svg+xml;charset=utf-8',
  })
  const sourceUrl = URL.createObjectURL(sourceBlob)

  try {
    const loaded = await new Promise<boolean>((resolve) => {
      image.onload = () => resolve(true)
      image.onerror = () => resolve(false)
      image.src = sourceUrl
    })
    if (!loaded) return undefined

    context.setTransform(1, 0, 0, 1, 0, 0)
    context.clearRect(0, 0, serializedSvg.width, serializedSvg.height)
    if (exportStyle.backgroundMode === 'opaque') {
      context.fillStyle = exportStyle.backgroundColor
      context.fillRect(0, 0, serializedSvg.width, serializedSvg.height)
    }
    context.drawImage(image, 0, 0, serializedSvg.width, serializedSvg.height)

    return context
  } finally {
    URL.revokeObjectURL(sourceUrl)
  }
}

async function rasterizeSvg({
  exportStyle,
  legendItems,
  svg,
  type,
}: {
  exportStyle: ExportStyle
  legendItems: Array<ExportLegendItem>
  svg: SVGSVGElement
  type: 'image/png' | 'image/jpeg'
}) {
  const serializedSvg = serializeSvg({ exportStyle, legendItems, svg })
  if (!serializedSvg) return undefined

  const ownerDocument = svg.ownerDocument
  const defaultView = ownerDocument.defaultView
  const ImageConstructor = defaultView?.Image
  if (!ImageConstructor) return undefined

  const image = new ImageConstructor()
  const sourceBlob = new Blob([serializedSvg.source], {
    type: 'image/svg+xml;charset=utf-8',
  })
  const sourceUrl = URL.createObjectURL(sourceBlob)

  try {
    const loaded = await new Promise<boolean>((resolve) => {
      image.onload = () => resolve(true)
      image.onerror = () => resolve(false)
      image.src = sourceUrl
    })
    if (!loaded) return undefined

    const scale = defaultView.devicePixelRatio || 1
    const canvas = ownerDocument.createElement('canvas')
    canvas.width = serializedSvg.width * scale
    canvas.height = serializedSvg.height * scale

    const context = canvas.getContext('2d')
    if (!context) return undefined

    context.scale(scale, scale)
    if (exportStyle.backgroundMode === 'opaque') {
      context.fillStyle = exportStyle.backgroundColor
      context.fillRect(0, 0, serializedSvg.width, serializedSvg.height)
    }
    context.drawImage(image, 0, 0, serializedSvg.width, serializedSvg.height)

    return await canvasToBlob({ canvas, type })
  } finally {
    URL.revokeObjectURL(sourceUrl)
  }
}

function getSupportedWebmMimeType(defaultView: Window) {
  const MediaRecorderConstructor = defaultView.MediaRecorder
  if (!MediaRecorderConstructor) return undefined

  return (
    ['video/webm;codecs=vp9', 'video/webm;codecs=vp8', 'video/webm'].find(
      (mimeType) => MediaRecorderConstructor.isTypeSupported(mimeType),
    ) ?? undefined
  )
}

async function encodeGifFrames({
  exportStyle,
  frames,
  ownerDocument,
}: {
  exportStyle: ExportStyle
  frames: Array<SerializedAnimationFrame>
  ownerDocument: Document
}) {
  if (!frames.length) return undefined

  const canvas = ownerDocument.createElement('canvas')
  const gif = GIFEncoder()

  for (const frame of frames) {
    const context = await drawSerializedSvgToCanvas({
      canvas,
      exportStyle,
      ownerDocument,
      serializedSvg: frame,
    })
    if (!context) return undefined

    const imageData = context.getImageData(0, 0, frame.width, frame.height)
    const palette = quantize(imageData.data, 256)
    const index = applyPalette(imageData.data, palette)
    gif.writeFrame(index, frame.width, frame.height, {
      delay: frame.delayMs,
      palette,
    })
  }

  gif.finish()
  const bytes = gif.bytes()
  const buffer = new ArrayBuffer(bytes.byteLength)
  new Uint8Array(buffer).set(bytes)

  return new Blob([buffer], { type: 'image/gif' })
}

async function encodeWebmFrames({
  exportStyle,
  frames,
  ownerDocument,
}: {
  exportStyle: ExportStyle
  frames: Array<SerializedAnimationFrame>
  ownerDocument: Document
}) {
  const defaultView = ownerDocument.defaultView
  if (!defaultView || !frames.length) return undefined

  const mimeType = getSupportedWebmMimeType(defaultView)
  const MediaRecorderConstructor = defaultView.MediaRecorder
  if (!mimeType || !MediaRecorderConstructor) return undefined

  const canvas = ownerDocument.createElement('canvas')
  const firstFrame = frames[0]
  if (!firstFrame) return undefined
  if (typeof canvas.captureStream !== 'function') return undefined

  canvas.width = firstFrame.width
  canvas.height = firstFrame.height

  const stream = canvas.captureStream(
    1000 / Math.max(animatedExportMinDelayMs, firstFrame.delayMs),
  )
  const recorder = new MediaRecorderConstructor(stream, { mimeType })
  const chunks: Array<Blob> = []
  const stopped = new Promise<Blob | undefined>((resolve) => {
    recorder.ondataavailable = (event) => {
      if (event.data.size > 0) {
        chunks.push(event.data)
      }
    }
    recorder.onerror = () => resolve(undefined)
    recorder.onstop = () => {
      resolve(chunks.length ? new Blob(chunks, { type: mimeType }) : undefined)
    }
  })
  const videoTrack = stream.getVideoTracks()[0]
  const requestFrame = videoTrack
    ? Reflect.get(videoTrack, 'requestFrame')
    : undefined
  let failed = false

  recorder.start()

  try {
    for (const frame of frames) {
      const context = await drawSerializedSvgToCanvas({
        canvas,
        exportStyle,
        ownerDocument,
        serializedSvg: frame,
      })
      if (!context) {
        failed = true
        break
      }

      if (typeof requestFrame === 'function' && videoTrack) {
        requestFrame.call(videoTrack)
      }
      await waitForMs(defaultView, frame.delayMs)
    }
  } finally {
    if (recorder.state !== 'inactive') {
      recorder.stop()
    }
  }

  const blob = await stopped
  stream.getTracks().forEach((track) => track.stop())

  return failed ? undefined : blob
}

function getSnapshotAnimationFrames({
  barOrientation,
  barSort,
  binType,
  bucketOffsetBounds,
  chartType,
  exportStyle,
  gradientIdPrefix,
  height,
  legendItems,
  ownerDocument,
  packages,
  playbackIntervalMs,
  queryData,
  width,
}: {
  barOrientation: BarOrientation
  barSort: LatestBarSort
  binType: BinType
  bucketOffsetBounds: LatestBucketOffsetBounds
  chartType: ChartType
  exportStyle: ExportStyle
  gradientIdPrefix: string
  height: number
  legendItems: Array<ExportLegendItem>
  ownerDocument: Document
  packages: Array<PackageGroup>
  playbackIntervalMs: number | undefined
  queryData: NpmQueryData
  width: number
}) {
  const frameDelayMs = getAnimatedExportFrameDelayMs(playbackIntervalMs)

  return getAnimatedExportBucketOffsets(bucketOffsetBounds)
    .map((bucketOffset): SerializedAnimationFrame | undefined => {
      const bucketQueryData = selectLatestBucketQueryData({
        queryData,
        binType,
        bucketOffset,
      })
      if (!bucketQueryData) return undefined

      const svg = createLatestBarExportChart({
        barOrientation,
        barSort,
        binType,
        chartType,
        gradientIdPrefix: `${gradientIdPrefix}-frame-${bucketOffset}`,
        height,
        ownerDocument,
        packages,
        queryData: bucketQueryData,
        width,
      })
      const serializedSvg = svg
        ? serializeSvg({ exportStyle, legendItems, svg })
        : undefined
      if (!serializedSvg) return undefined

      return {
        ...serializedSvg,
        delayMs: frameDelayMs,
      }
    })
    .filter((frame): frame is SerializedAnimationFrame => frame !== undefined)
}

function ChartActions({
  canExportAnimation,
  disabled,
  embedConfig,
  exportingFormat,
  onExport,
  onToggleLegend,
  showLegend,
}: {
  canExportAnimation: boolean
  disabled: boolean
  embedConfig?: NpmStatsChartEmbedConfig
  exportingFormat: ChartExportFormat | null
  onExport: (format: ChartExportFormat) => void
  onToggleLegend: () => void
  showLegend: boolean
}) {
  const buttonStyles = twMerge(
    chartActionButtonStyles,
    disabled && 'cursor-not-allowed opacity-50',
  )

  return (
    <div className="flex items-center gap-1">
      <DropdownMenu>
        <Tooltip
          content={
            exportingFormat
              ? `Exporting ${getExportFileName(exportingFormat)}`
              : disabled
                ? 'Chart export available after render'
                : 'Export chart'
          }
        >
          <DropdownMenuTrigger asChild>
            <button
              aria-label="Export chart"
              className={buttonStyles}
              disabled={disabled}
              type="button"
            >
              <DownloadIcon className="size-3" />
            </button>
          </DropdownMenuTrigger>
        </Tooltip>
        <DropdownMenuContent
          className={chartActionDropdownContentStyles}
          collisionPadding={8}
          sideOffset={5}
        >
          <div className="mb-1 flex items-center justify-between px-0.5 text-xs font-medium">
            <span>Export</span>
          </div>
          {chartExportOptions
            .filter(
              ({ value }) =>
                canExportAnimation || !isAnimatedExportFormat(value),
            )
            .map(({ label, value }) => (
              <DropdownMenuItem
                className={chartActionDropdownItemStyles}
                disabled={disabled}
                key={value}
                onSelect={() => onExport(value)}
              >
                {label}
              </DropdownMenuItem>
            ))}
        </DropdownMenuContent>
      </DropdownMenu>
      <Tooltip
        content={
          disabled
            ? 'Chart legend available after render'
            : showLegend
              ? 'Hide chart legend'
              : 'Show chart legend'
        }
      >
        <button
          aria-label={showLegend ? 'Hide chart legend' : 'Show chart legend'}
          aria-pressed={showLegend}
          className={twMerge(
            buttonStyles,
            showLegend && 'bg-blue-500/10 text-blue-500',
          )}
          disabled={disabled}
          onClick={onToggleLegend}
          type="button"
        >
          <ListIcon className="size-3" />
        </button>
      </Tooltip>
      {embedConfig ? <EmbedChartAction embedConfig={embedConfig} /> : null}
    </div>
  )
}

function getAbsoluteEmbedUrl(url: string) {
  const base =
    typeof window === 'undefined'
      ? 'https://tanstack.com'
      : window.location.origin

  return new URL(url, base).toString()
}

function CopyButton({
  copied,
  label,
  onCopy,
}: {
  copied: boolean
  label: string
  onCopy: () => void
}) {
  return (
    <button
      className="flex shrink-0 items-center gap-1 rounded bg-gray-500/10 px-2 py-1 text-[11px] font-medium text-gray-600 hover:bg-gray-500/20 hover:text-blue-500 dark:text-gray-300 dark:hover:text-gray-100"
      onClick={onCopy}
      type="button"
    >
      {copied ? (
        <CheckIcon className="size-3" />
      ) : (
        <CopyIcon className="size-3" />
      )}
      {copied ? 'Copied' : label}
    </button>
  )
}

function EmbedOption({
  checked,
  children,
  disabled,
  onCheckedChange,
}: {
  checked: boolean
  children: React.ReactNode
  disabled?: boolean
  onCheckedChange: (checked: boolean) => void
}) {
  return (
    <label
      className={twMerge(
        'flex items-center gap-2 text-xs text-gray-700 dark:text-gray-200',
        disabled && 'cursor-not-allowed opacity-50',
      )}
    >
      <input
        checked={checked}
        className="size-3.5 accent-blue-500"
        disabled={disabled}
        onChange={(event) => onCheckedChange(event.currentTarget.checked)}
        type="checkbox"
      />
      <span>{children}</span>
    </label>
  )
}

function EmbedChartAction({
  embedConfig,
}: {
  embedConfig: NpmStatsChartEmbedConfig
}) {
  const [showLegend, setShowLegend] = React.useState(true)
  const [includeTimelineRange, setIncludeTimelineRange] = React.useState(
    embedConfig.hasTimelineRange,
  )
  const [lockWidth, setLockWidth] = React.useState(false)
  const [iframeHeight, setIframeHeight] = React.useState(420)
  const [copiedTarget, setCopiedTarget] = React.useState<
    'iframe' | 'url' | null
  >(null)

  React.useEffect(() => {
    if (!embedConfig.hasTimelineRange) {
      setIncludeTimelineRange(false)
    }
  }, [embedConfig.hasTimelineRange])

  React.useEffect(() => {
    if (!embedConfig.hasWidth) {
      setLockWidth(false)
    }
  }, [embedConfig.hasWidth])

  const embedUrl = getAbsoluteEmbedUrl(
    embedConfig.buildUrl({
      includeTimelineRange,
      lockWidth,
      showLegend,
    }),
  )
  const iframeCode = `<iframe src="${embedUrl}" title="TanStack NPM Stats chart" loading="lazy" style="width:100%;height:${iframeHeight}px;border:0;"></iframe>`

  const copyText = React.useCallback(
    async (target: 'iframe' | 'url', text: string) => {
      await copyTextToClipboard(text)
      setCopiedTarget(target)
      window.setTimeout(() => {
        setCopiedTarget((currentTarget) =>
          currentTarget === target ? null : currentTarget,
        )
      }, 1500)
    },
    [],
  )

  return (
    <DropdownMenu>
      <Tooltip content="Embed chart">
        <DropdownMenuTrigger asChild>
          <button
            aria-label="Embed chart"
            className={chartActionButtonStyles}
            type="button"
          >
            <CodeIcon className="size-3" />
          </button>
        </DropdownMenuTrigger>
      </Tooltip>
      <DropdownMenuContent
        align="end"
        className="z-50 w-[min(420px,calc(100vw-2rem))] rounded-md bg-white p-3 text-gray-900 shadow-lg dark:bg-gray-800 dark:text-gray-100"
        collisionPadding={8}
        sideOffset={5}
      >
        <div className="space-y-3">
          <div className="flex items-center justify-between gap-3">
            <div className="text-xs font-medium">Embed chart</div>
            <label className="flex items-center gap-2 text-xs text-gray-600 dark:text-gray-300">
              <span>Iframe height</span>
              <input
                className="w-16 rounded border border-gray-500/20 bg-gray-50 px-1.5 py-1 text-right font-mono text-[11px] outline-none focus:border-blue-500 dark:bg-gray-900"
                max={1200}
                min={240}
                onChange={(event) => {
                  const nextHeight = Number(event.currentTarget.value)
                  if (!Number.isFinite(nextHeight)) return
                  setIframeHeight(Math.max(240, Math.min(1200, nextHeight)))
                }}
                type="number"
                value={iframeHeight}
              />
            </label>
          </div>

          <div className="grid gap-1.5">
            <EmbedOption checked={showLegend} onCheckedChange={setShowLegend}>
              Show legend by default
            </EmbedOption>
            <EmbedOption
              checked={includeTimelineRange}
              disabled={!embedConfig.hasTimelineRange}
              onCheckedChange={setIncludeTimelineRange}
            >
              Include current timeline zoom
            </EmbedOption>
            <EmbedOption
              checked={lockWidth}
              disabled={!embedConfig.hasWidth}
              onCheckedChange={setLockWidth}
            >
              Lock current chart width
            </EmbedOption>
          </div>

          <div className="space-y-1.5">
            <div className="flex items-center justify-between gap-2">
              <span className="text-[11px] font-medium text-gray-500">URL</span>
              <CopyButton
                copied={copiedTarget === 'url'}
                label="Copy URL"
                onCopy={() => {
                  void copyText('url', embedUrl)
                }}
              />
            </div>
            <input
              className={chartEmbedInputStyles}
              readOnly
              value={embedUrl}
            />
          </div>

          <div className="space-y-1.5">
            <div className="flex items-center justify-between gap-2">
              <span className="text-[11px] font-medium text-gray-500">
                Iframe
              </span>
              <CopyButton
                copied={copiedTarget === 'iframe'}
                label="Copy iframe"
                onCopy={() => {
                  void copyText('iframe', iframeCode)
                }}
              />
            </div>
            <textarea
              className={twMerge(chartEmbedInputStyles, 'min-h-20 resize-none')}
              readOnly
              value={iframeCode}
            />
          </div>
        </div>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}

function useElementWidth() {
  const containerRef = React.useRef<HTMLDivElement>(null)
  const [width, setWidth] = React.useState(0)

  React.useEffect(() => {
    const container = containerRef.current
    if (!container) return

    const setMeasuredWidth = (nextWidth: number) => {
      setWidth((previousWidth) =>
        previousWidth === nextWidth ? previousWidth : nextWidth,
      )
    }

    const updateWidth = () => {
      setMeasuredWidth(Math.round(container.getBoundingClientRect().width))
    }

    updateWidth()

    const ownerWindow = container.ownerDocument.defaultView
    if (!ownerWindow?.ResizeObserver) {
      ownerWindow?.addEventListener('resize', updateWidth)
      return () => {
        ownerWindow?.removeEventListener('resize', updateWidth)
      }
    }

    const resizeObserver = new ownerWindow.ResizeObserver((entries) => {
      const entry = entries[0]
      if (!entry) return

      setMeasuredWidth(Math.round(entry.contentRect.width))
    })

    resizeObserver.observe(container)

    return () => {
      resizeObserver.disconnect()
    }
  }, [])

  return { containerRef, width }
}

function useMeasuredElementHeight<T extends HTMLElement>() {
  const [element, setElement] = React.useState<T | null>(null)
  const [height, setHeight] = React.useState(0)

  const ref = React.useCallback((nextElement: T | null) => {
    setElement(nextElement)
  }, [])

  React.useEffect(() => {
    if (!element) {
      setHeight(0)
      return
    }

    const setMeasuredHeight = (nextHeight: number) => {
      setHeight((previousHeight) =>
        previousHeight === nextHeight ? previousHeight : nextHeight,
      )
    }

    const updateHeight = () => {
      setMeasuredHeight(Math.ceil(element.getBoundingClientRect().height))
    }

    updateHeight()

    const ownerWindow = element.ownerDocument.defaultView
    if (!ownerWindow?.ResizeObserver) {
      ownerWindow?.addEventListener('resize', updateHeight)
      return () => {
        ownerWindow?.removeEventListener('resize', updateHeight)
      }
    }

    const resizeObserver = new ownerWindow.ResizeObserver(updateHeight)
    resizeObserver.observe(element)

    return () => {
      resizeObserver.disconnect()
    }
  }, [element])

  return { height, ref }
}

function TimelineRangeScrubber({
  dates,
  endIndex,
  marginLeft,
  marginRight,
  onRangeChange,
  startIndex,
}: TimelineRangeScrubberProps) {
  const scrubberRef = React.useRef<HTMLDivElement>(null)
  const [dragState, setDragState] =
    React.useState<TimelineScrubberDragState | null>(null)
  const [panState, setPanState] =
    React.useState<TimelineScrubberPanState | null>(null)
  const maxIndex = dates.length - 1
  if (maxIndex < 1) return null

  const safeStartIndex = Math.min(
    clampTimelineIndex(startIndex, maxIndex),
    maxIndex - 1,
  )
  const safeEndIndex = Math.max(
    clampTimelineIndex(endIndex, maxIndex),
    safeStartIndex + 1,
  )
  const displayStartIndex = dragState
    ? Math.min(dragState.anchorIndex, dragState.focusIndex)
    : safeStartIndex
  const displayEndIndex = dragState
    ? Math.max(dragState.anchorIndex, dragState.focusIndex)
    : safeEndIndex
  const startPercent = (displayStartIndex / maxIndex) * 100
  const endPercent = (displayEndIndex / maxIndex) * 100
  const middlePercent = (startPercent + endPercent) / 2
  const startLabel = formatTimelineScrubberDate(dates[safeStartIndex])
  const endLabel = formatTimelineScrubberDate(dates[safeEndIndex])
  const isZoomed = safeStartIndex > 0 || safeEndIndex < maxIndex

  const handleStartChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const nextStartIndex = Math.min(
      Number(event.currentTarget.value),
      safeEndIndex - 1,
    )
    onRangeChange(nextStartIndex, safeEndIndex)
  }

  const handleEndChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const nextEndIndex = Math.max(
      Number(event.currentTarget.value),
      safeStartIndex + 1,
    )
    onRangeChange(safeStartIndex, nextEndIndex)
  }
  const handleSelectionPointerDown = (
    event: React.PointerEvent<HTMLDivElement>,
  ) => {
    if (event.button !== 0 || event.target instanceof HTMLInputElement) return

    event.preventDefault()
    const index = getTimelineIndexFromPointer({
      clientX: event.clientX,
      element: event.currentTarget,
      maxIndex,
    })

    event.currentTarget.setPointerCapture(event.pointerId)
    setDragState({ anchorIndex: index, focusIndex: index })
  }
  const handleSelectionPointerMove = (
    event: React.PointerEvent<HTMLDivElement>,
  ) => {
    if (!dragState || panState) return

    const focusIndex = getTimelineIndexFromPointer({
      clientX: event.clientX,
      element: event.currentTarget,
      maxIndex,
    })

    setDragState({ ...dragState, focusIndex })
  }
  const handleSelectionPointerUp = (
    event: React.PointerEvent<HTMLDivElement>,
  ) => {
    if (!dragState || panState) return

    if (event.currentTarget.hasPointerCapture(event.pointerId)) {
      event.currentTarget.releasePointerCapture(event.pointerId)
    }

    const focusIndex = getTimelineIndexFromPointer({
      clientX: event.clientX,
      element: event.currentTarget,
      maxIndex,
    })
    const start = Math.min(dragState.anchorIndex, focusIndex)
    const end = Math.max(dragState.anchorIndex, focusIndex)
    setDragState(null)

    if (start === end) return

    onRangeChange(start, end)
  }
  const handleSelectionPointerCancel = (
    event: React.PointerEvent<HTMLDivElement>,
  ) => {
    if (event.currentTarget.hasPointerCapture(event.pointerId)) {
      event.currentTarget.releasePointerCapture(event.pointerId)
    }
    setDragState(null)
    setPanState(null)
  }
  const handleSelectionDoubleClick = (
    event: React.MouseEvent<HTMLDivElement>,
  ) => {
    event.preventDefault()
    setDragState(null)
    setPanState(null)
    onRangeChange(0, maxIndex)
  }
  const handlePanPointerDown = (
    event: React.PointerEvent<HTMLButtonElement>,
  ) => {
    const scrubber = scrubberRef.current
    if (event.button !== 0 || !scrubber) return

    event.preventDefault()
    event.stopPropagation()
    const anchorIndex = getTimelineIndexFromPointer({
      clientX: event.clientX,
      element: scrubber,
      maxIndex,
    })

    event.currentTarget.setPointerCapture(event.pointerId)
    setPanState({
      anchorIndex,
      endIndex: safeEndIndex,
      startIndex: safeStartIndex,
    })
  }
  const handlePanPointerMove = (
    event: React.PointerEvent<HTMLButtonElement>,
  ) => {
    const scrubber = scrubberRef.current
    if (!panState || !scrubber) return

    event.preventDefault()
    event.stopPropagation()
    const focusIndex = getTimelineIndexFromPointer({
      clientX: event.clientX,
      element: scrubber,
      maxIndex,
    })
    const delta = focusIndex - panState.anchorIndex
    const rangeSize = panState.endIndex - panState.startIndex
    const nextStartIndex = Math.max(
      0,
      Math.min(panState.startIndex + delta, maxIndex - rangeSize),
    )
    const nextEndIndex = nextStartIndex + rangeSize

    if (nextStartIndex !== safeStartIndex || nextEndIndex !== safeEndIndex) {
      onRangeChange(nextStartIndex, nextEndIndex)
    }
  }
  const handlePanPointerUp = (event: React.PointerEvent<HTMLButtonElement>) => {
    event.stopPropagation()
    if (event.currentTarget.hasPointerCapture(event.pointerId)) {
      event.currentTarget.releasePointerCapture(event.pointerId)
    }
    setPanState(null)
  }
  const handlePanPointerCancel = (
    event: React.PointerEvent<HTMLButtonElement>,
  ) => {
    event.stopPropagation()
    if (event.currentTarget.hasPointerCapture(event.pointerId)) {
      event.currentTarget.releasePointerCapture(event.pointerId)
    }
    setPanState(null)
  }

  return (
    <div
      className="group relative h-8 cursor-crosshair"
      onDoubleClick={handleSelectionDoubleClick}
      onPointerCancel={handleSelectionPointerCancel}
      onPointerDown={handleSelectionPointerDown}
      onPointerMove={handleSelectionPointerMove}
      onPointerUp={handleSelectionPointerUp}
      ref={scrubberRef}
      style={{ marginLeft, marginRight }}
    >
      <div className="absolute left-0 right-0 top-1/2 h-3 -translate-y-1/2 cursor-crosshair rounded-sm bg-gray-500/10 dark:bg-gray-100/10" />
      <div
        className="absolute top-1/2 h-3 -translate-y-1/2 cursor-crosshair rounded-sm bg-gray-500/20 dark:bg-gray-100/20"
        style={{
          left: `${startPercent}%`,
          right: `${100 - endPercent}%`,
        }}
      />
      {isZoomed ? (
        <button
          aria-label="Pan timeline zoom range"
          className={twMerge(
            'absolute top-1/2 z-10 hidden h-3 w-6 -translate-x-1/2 -translate-y-1/2 cursor-grab items-center justify-center gap-0.5 rounded-sm border border-gray-400 bg-gray-300 text-gray-700 shadow-xs group-hover:flex dark:border-gray-600 dark:bg-gray-700 dark:text-gray-200',
            panState && 'flex cursor-grabbing',
          )}
          onPointerCancel={handlePanPointerCancel}
          onPointerDown={handlePanPointerDown}
          onPointerMove={handlePanPointerMove}
          onPointerUp={handlePanPointerUp}
          style={{ left: `${middlePercent}%` }}
          type="button"
        >
          <span className="h-2 w-px bg-current opacity-80" />
          <span className="h-2 w-px bg-current opacity-80" />
        </button>
      ) : null}
      <input
        aria-label="Timeline zoom start"
        aria-valuetext={startLabel}
        className={twMerge(
          timelineScrubberInputStyles,
          timelineScrubberStartInputStyles,
          dragState && 'hidden',
        )}
        max={maxIndex}
        min={0}
        onChange={handleStartChange}
        step={1}
        type="range"
        value={safeStartIndex}
      />
      <input
        aria-label="Timeline zoom end"
        aria-valuetext={endLabel}
        className={twMerge(
          timelineScrubberInputStyles,
          timelineScrubberEndInputStyles,
          dragState && 'hidden',
        )}
        max={maxIndex}
        min={0}
        onChange={handleEndChange}
        step={1}
        type="range"
        value={safeEndIndex}
      />
    </div>
  )
}

type NpmStatsChartInput = {
  barOrientation: BarOrientation
  bars: Array<NpmBarPoint>
  chartType: ChartType
  gradients: Array<ChartLinearGradient>
  historyComplete: Array<NpmHistoryPoint>
  historyPartial: Array<NpmHistoryPoint>
  marginBottom: number
  marginLeft: number
  marginRight: number
  stackedHistory: Array<NpmStackedHistoryPoint>
  timelineDomain?: [Date, Date]
  viewMode: ViewMode
  xLabel?: string
  yFormat?: (value: number) => string
  yLabel?: string
}

function createNpmStatsChart(input: NpmStatsChartInput) {
  const definition = defineChart(() => createNpmStatsChartSpec(input))

  return defineChart(definition, {
    svgAnimation: {
      duration: chartUpdateTransitionDurationMs,
      easing: 'ease-out',
    },
    focus:
      input.viewMode === 'latest' && input.barOrientation === 'horizontal'
        ? 'group-y'
        : 'group-x',
    tooltip: {
      use: tooltip,
      anchor: 'pointer',
      content: (points, context) =>
        getNpmChartTooltipContent(points, context, input),
      placement: ['top', 'right', 'left', 'bottom'],
    },
  })
}

function createNpmStatsChartSpec(input: NpmStatsChartInput) {
  const common = {
    gradients: input.gradients,
    margin: {
      top: 20,
      right: input.marginRight,
      bottom: input.marginBottom,
      left: input.marginLeft,
    },
    theme: { background: 'transparent' },
  }

  if (input.viewMode === 'latest') {
    const categories = input.bars.map((bar) => bar.category)
    const categoryDomain = [...new Set(categories)]
    const maximum = d3.max(input.bars, (bar) => bar.value2) ?? 1
    const categoricalScale = d3
      .scaleBand<string>()
      .domain(categoryDomain)
      .paddingInner(0.1)
      .paddingOuter(0.05)
    const numericScale = d3.scaleLinear().domain([0, maximum]).nice(7)

    if (input.barOrientation === 'horizontal') {
      return {
        ...common,
        marks: [
          ruleX([0], {
            stroke: 'currentColor',
            strokeWidth: 1.5,
            strokeOpacity: 0.5,
          }),
          barX(input.bars, {
            id: 'npm-latest-bars-x',
            x1: 'value1',
            x2: 'value2',
            y: 'category',
            z: 'seriesName',
            key: 'id',
            fill: (datum) => datum.fill,
            inset: input.chartType === 'stacked-bar' ? 1 : 2,
            radius: input.chartType === 'stacked-bar' ? undefined : 2,
          }),
        ],
        scales: {
          x: {
            scale: numericScale,
            axis: {
              label: input.xLabel,
              ticks: { format: formatCompactAxisNumber },
            },
            grid: true,
          },
          y: { scale: categoricalScale },
        },
      }
    }

    return {
      ...common,
      marks: [
        ruleY([0], {
          stroke: 'currentColor',
          strokeWidth: 1.5,
          strokeOpacity: 0.5,
        }),
        barY(input.bars, {
          id: 'npm-latest-bars-y',
          x: 'category',
          y1: 'value1',
          y2: 'value2',
          z: 'seriesName',
          key: 'id',
          fill: (datum) => datum.fill,
          inset: input.chartType === 'stacked-bar' ? 1 : 2,
          radius: input.chartType === 'stacked-bar' ? undefined : 2,
        }),
      ],
      scales: {
        x: { scale: categoricalScale },
        y: {
          scale: numericScale,
          axis: {
            label: input.yLabel,
            ticks: { format: input.yFormat },
          },
          grid: true,
        },
      },
    }
  }

  const allDates = [
    ...input.historyComplete.map((point) => point.date),
    ...input.historyPartial.map((point) => point.date),
    ...input.stackedHistory.map((point) => point.date),
  ]
  const dateExtent = d3.extent(allDates)
  const dateDomain =
    input.timelineDomain ??
    (dateExtent[0] && dateExtent[1]
      ? [dateExtent[0], dateExtent[1]]
      : [new Date(0), new Date(86_400_000)])
  const x = {
    scale: d3.scaleUtc().domain(dateDomain),
    axis: { label: input.xLabel },
    grid: true,
  }
  const curve = d3Curve(d3.curveMonotoneX)

  if (input.chartType === 'line') {
    const values = [
      ...input.historyComplete.map((point) => point.value),
      ...input.historyPartial.map((point) => point.value),
    ]
    const valueExtent = d3.extent([...values, 0])

    return {
      ...common,
      marks: [
        ruleY([0], {
          stroke: 'currentColor',
          strokeWidth: 1.5,
          strokeOpacity: 0.5,
        }),
        lineY(input.historyPartial, {
          id: 'npm-history-partial',
          x: 'date',
          y: 'value',
          z: 'seriesName',
          key: 'id',
          stroke: (datum) => datum.color,
          strokeWidth: 1.5,
          strokeOpacity: 0.8,
          strokeDasharray: '2 4',
          curve,
        }),
        lineY(input.historyComplete, {
          id: 'npm-history-complete',
          x: 'date',
          y: 'value',
          z: 'seriesName',
          key: 'id',
          stroke: (datum) => datum.color,
          strokeWidth: 2,
          curve,
        }),
      ],
      scales: {
        x,
        y: {
          scale: d3
            .scaleLinear()
            .domain([valueExtent[0] ?? 0, valueExtent[1] ?? 1])
            .nice(5),
          axis: {
            label: input.yLabel,
            ticks: { format: input.yFormat },
          },
          grid: true,
        },
      },
      clip: !!input.timelineDomain,
    }
  }

  const stackedValues = input.stackedHistory.flatMap((point) => [
    point.value1,
    point.value2,
  ])
  const stackedExtent = d3.extent([...stackedValues, 0])

  return {
    ...common,
    marks: [
      ruleY([0], {
        stroke: 'currentColor',
        strokeWidth: 1.5,
        strokeOpacity: 0.5,
      }),
      areaY(input.stackedHistory, {
        id: `npm-history-${input.chartType}-areas`,
        x: 'date',
        y1: 'value1',
        y2: 'value2',
        z: 'seriesName',
        key: 'id',
        fill: (datum) => datum.fill,
        curve,
      }),
      lineY(input.stackedHistory, {
        id: `npm-history-${input.chartType}-caps`,
        x: 'date',
        y: 'value2',
        z: 'seriesName',
        key: 'id',
        stroke: (datum) => datum.color,
        strokeWidth: 1.4,
        strokeOpacity: 0.95,
        curve,
      }),
    ],
    scales: {
      x,
      y: {
        scale: d3
          .scaleLinear()
          .domain([stackedExtent[0] ?? 0, stackedExtent[1] ?? 1])
          .nice(5),
        axis: {
          label: input.yLabel,
          ticks: { format: input.yFormat },
        },
        grid: true,
      },
    },
    clip: !!input.timelineDomain,
  }
}

type NpmStatsChartDefinitionRevision = {
  barOrientation: BarOrientation
  barSort: LatestBarSort
  binType: BinType
  chartType: ChartType
  gradientIdPrefix: string
  height: number
  normalizeBaseline: boolean
  packages: PackageGroup[]
  queryData: NpmQueryData | undefined
  range: TimeRange
  showBaseline: boolean
  showDataMode: ShowDataMode
  timelineEnd: number | undefined
  timelineStart: number | undefined
  today: number
  transform: TransformMode
  viewMode: ViewMode
  width: number
}

function useNpmStatsChartDefinition(
  input: NpmStatsChartInput,
  revision: NpmStatsChartDefinitionRevision,
) {
  const cached = React.useRef<{
    definition: ReturnType<typeof createNpmStatsChart>
    revision: NpmStatsChartDefinitionRevision
  }>(undefined)

  if (!cached.current || cached.current.revision !== revision) {
    cached.current = {
      definition: createNpmStatsChart(input),
      revision,
    }
  }

  return cached.current.definition
}

function ChartFigure({
  colorLegendEntries,
  definitionRevision,
  footer,
  height,
  input,
  legendRef,
  onRenderedChange,
  svgRef,
  showLegend,
  width,
}: {
  colorLegendEntries: Array<ColorLegendEntry>
  definitionRevision: NpmStatsChartDefinitionRevision
  footer?: React.ReactNode
  height: number
  input: NpmStatsChartInput
  legendRef: { current: HTMLDivElement | null }
  onRenderedChange: (rendered: boolean) => void
  svgRef: { current: SVGSVGElement | null }
  showLegend: boolean
  width: number
}) {
  const { height: legendHeight, ref: measuredLegendRef } =
    useMeasuredElementHeight<HTMLDivElement>()
  const { height: footerHeight, ref: footerRef } =
    useMeasuredElementHeight<HTMLDivElement>()
  const setLegendRef = React.useCallback(
    (element: HTMLDivElement | null) => {
      legendRef.current = element
      measuredLegendRef(element)
    },
    [legendRef, measuredLegendRef],
  )
  const chartHeight = getChartContentHeight({
    reservedHeight: legendHeight + footerHeight,
    totalHeight: height,
  })
  const definition = useNpmStatsChartDefinition(input, definitionRevision)

  React.useEffect(() => {
    return () => {
      svgRef.current = null
      onRenderedChange(false)
    }
  }, [onRenderedChange, svgRef])

  return (
    <div className="relative" style={{ height }}>
      <div
        className={twMerge(
          'flex flex-wrap justify-center gap-x-3 gap-y-1 overflow-x-auto px-2 pb-2 text-xs',
          !showLegend && 'hidden',
        )}
        ref={setLegendRef}
      >
        {colorLegendEntries.map((entry) => (
          <span
            className="flex items-center gap-1.5"
            data-chart-legend-item=""
            data-color={entry.color}
            key={entry.label}
          >
            <span
              className="size-2.5 rounded-sm"
              style={{ backgroundColor: entry.color }}
            />
            {entry.label}
          </span>
        ))}
      </div>
      <Chart
        definition={definition}
        width={width}
        height={chartHeight}
        ariaLabel={
          input.viewMode === 'latest'
            ? 'npm download totals by package'
            : 'npm downloads by date'
        }
        renderSvg={renderChartSvgWithResources}
        onRender={({ svg }) => {
          svgRef.current = svg
          onRenderedChange(true)
        }}
      />
      {footer ? (
        <div className="flow-root pt-1" ref={footerRef}>
          {footer}
        </div>
      ) : null}
    </div>
  )
}

// Props for the NPMStatsChart component
export type NPMStatsChartProps = {
  actionsContainer?: HTMLElement | null
  embedConfig?: NpmStatsChartEmbedConfig
  onActionsMountedChange?: (mounted: boolean) => void
  queryData: NpmQueryData | undefined
  height: number
  viewMode: ViewMode
  chartType: ChartType
  barSort: LatestBarSort
  barOrientation: BarOrientation
  transform: TransformMode
  binType: BinType
  packages: PackageGroup[]
  range: TimeRange
  showDataMode: ShowDataMode
  normalizeBaseline?: boolean
  showBaseline?: boolean
  showLegend?: boolean
  timelineEnd?: number
  timelineStart?: number
  animationBucketOffsetBounds?: LatestBucketOffsetBounds
  animationFrameIntervalMs?: number
  animationQueryData?: NpmQueryData
  onShowLegendChange?: (showLegend: boolean) => void
  onTimelineRangeChange?: (range: TimelineRangeValue) => void
}

export function NPMStatsChart({
  actionsContainer,
  embedConfig,
  onActionsMountedChange,
  queryData,
  height,
  viewMode,
  chartType,
  barSort,
  barOrientation,
  transform,
  binType,
  packages,
  range,
  showDataMode,
  normalizeBaseline = true,
  showBaseline = false,
  showLegend: controlledShowLegend,
  timelineEnd,
  timelineStart,
  animationBucketOffsetBounds,
  animationFrameIntervalMs,
  animationQueryData,
  onShowLegendChange,
  onTimelineRangeChange,
}: NPMStatsChartProps) {
  const { containerRef, width } = useElementWidth()
  const rawGradientId = React.useId()
  const gradientIdPrefix = `npm-stats-fill-${rawGradientId.replace(
    /[^a-zA-Z0-9_-]/g,
    '',
  )}`
  const now = getUtcToday()
  const today = now.getTime()
  const chartDefinitionRevision = React.useMemo(
    (): NpmStatsChartDefinitionRevision => ({
      barOrientation,
      barSort,
      binType,
      chartType,
      gradientIdPrefix,
      height,
      normalizeBaseline,
      packages,
      queryData,
      range,
      showBaseline,
      showDataMode,
      timelineEnd,
      timelineStart,
      today,
      transform,
      viewMode,
      width,
    }),
    [
      barOrientation,
      barSort,
      binType,
      chartType,
      gradientIdPrefix,
      height,
      normalizeBaseline,
      packages,
      queryData,
      range,
      showBaseline,
      showDataMode,
      timelineEnd,
      timelineStart,
      today,
      transform,
      viewMode,
      width,
    ],
  )
  const legendRef = React.useRef<HTMLDivElement>(null)
  const svgRef = React.useRef<SVGSVGElement | null>(null)
  const [uncontrolledShowLegend, setUncontrolledShowLegend] =
    React.useState(false)
  const showLegend = controlledShowLegend ?? uncontrolledShowLegend
  const showLegendRef = React.useRef(showLegend)
  const [chartRendered, setChartRendered] = React.useState(false)
  const [exportingFormat, setExportingFormat] =
    React.useState<ChartExportFormat | null>(null)

  React.useEffect(() => {
    showLegendRef.current = showLegend
  }, [showLegend])

  React.useEffect(() => {
    if (!actionsContainer) return

    onActionsMountedChange?.(true)
    return () => {
      onActionsMountedChange?.(false)
    }
  }, [actionsContainer, onActionsMountedChange])

  const handleChartRenderedChange = React.useCallback((rendered: boolean) => {
    setChartRendered((previousRendered) =>
      previousRendered === rendered ? previousRendered : rendered,
    )
  }, [])
  const canExportAnimation =
    viewMode === 'latest' &&
    !!animationQueryData?.length &&
    !!animationBucketOffsetBounds &&
    animationBucketOffsetBounds.minOffset <
      animationBucketOffsetBounds.maxOffset

  const handleExport = React.useCallback(
    async (format: ChartExportFormat) => {
      if (exportingFormat) return

      const svg = getChartSvg(svgRef.current)
      if (!svg) return

      setExportingFormat(format)

      try {
        const exportStyle = getExportStyle({ format, svg })
        const legendItems = getExportLegendItems({
          container: legendRef.current,
          showLegend: showLegendRef.current,
        })

        if (format === 'svg') {
          const serializedSvg = serializeSvg({ exportStyle, legendItems, svg })
          if (!serializedSvg) return

          downloadBlob({
            blob: new Blob([serializedSvg.source], {
              type: 'image/svg+xml;charset=utf-8',
            }),
            fileName: getExportFileName(format),
            ownerDocument: svg.ownerDocument,
          })
          return
        }

        if (isAnimatedExportFormat(format)) {
          if (
            !canExportAnimation ||
            !animationQueryData ||
            !animationBucketOffsetBounds
          ) {
            return
          }

          const frames = getSnapshotAnimationFrames({
            barOrientation,
            barSort,
            binType,
            bucketOffsetBounds: animationBucketOffsetBounds,
            chartType,
            exportStyle,
            gradientIdPrefix,
            height,
            legendItems,
            ownerDocument: svg.ownerDocument,
            packages,
            playbackIntervalMs: animationFrameIntervalMs,
            queryData: animationQueryData,
            width,
          })
          const blob =
            format === 'gif'
              ? await encodeGifFrames({
                  exportStyle,
                  frames,
                  ownerDocument: svg.ownerDocument,
                })
              : await encodeWebmFrames({
                  exportStyle,
                  frames,
                  ownerDocument: svg.ownerDocument,
                })
          if (!blob) return

          downloadBlob({
            blob,
            fileName: getExportFileName(format),
            ownerDocument: svg.ownerDocument,
          })
          return
        }

        const blob = await rasterizeSvg({
          exportStyle,
          legendItems,
          svg,
          type: format === 'png' ? 'image/png' : 'image/jpeg',
        })
        if (!blob) return

        downloadBlob({
          blob,
          fileName: getExportFileName(format),
          ownerDocument: svg.ownerDocument,
        })
      } finally {
        setExportingFormat(null)
      }
    },
    [
      animationBucketOffsetBounds,
      animationFrameIntervalMs,
      animationQueryData,
      barOrientation,
      barSort,
      binType,
      canExportAnimation,
      chartType,
      exportingFormat,
      gradientIdPrefix,
      height,
      packages,
      width,
    ],
  )

  const handleToggleLegend = React.useCallback(() => {
    const nextShowLegend = !showLegend

    if (controlledShowLegend === undefined) {
      setUncontrolledShowLegend(nextShowLegend)
    }

    onShowLegendChange?.(nextShowLegend)
  }, [controlledShowLegend, onShowLegendChange, showLegend])

  const canUseChartActions =
    chartRendered && !!queryData?.length && width > 0 && !exportingFormat
  const chartActions = actionsContainer
    ? createPortal(
        <ChartActions
          canExportAnimation={canExportAnimation}
          disabled={!canUseChartActions}
          embedConfig={embedConfig}
          exportingFormat={exportingFormat}
          onExport={(format) => {
            void handleExport(format)
          }}
          onToggleLegend={handleToggleLegend}
          showLegend={canUseChartActions && showLegend}
        />,
        actionsContainer,
      )
    : null

  if (!queryData?.length) {
    return (
      <div ref={containerRef} className="w-full" style={{ height }}>
        {chartActions}
      </div>
    )
  }

  const binOption = binningOptionsByType[binType]
  const binUnit = binningOptionsByType[binType].bin
  const effectiveTransform =
    viewMode === 'history' && chartType === 'line' ? transform : 'none'
  const useBaseline = viewMode === 'history'

  let startDate = (() => {
    if (viewMode === 'latest') {
      const earliestQueryStart = queryData
        .map((pkg) => getUtcDay(new Date(pkg.start)))
        .sort((a, b) => a.getTime() - b.getTime())[0]
      return earliestQueryStart || NPM_STATS_START_DATE
    }

    switch (range) {
      case 'all-time':
        // Use the actual start date from the query data, or fall back to npm's stats start date
        const earliestActualStartDate = queryData
          .map((pkg) => pkg.actualStartDate)
          .filter((d): d is Date => d !== undefined)
          .sort((a, b) => a.getTime() - b.getTime())[0]
        return earliestActualStartDate || NPM_STATS_START_DATE
      default:
        return getHistoryStartDate(range, now)
    }
  })()

  startDate = binOption.bin.floor(startDate)

  const combinedPackageGroups = queryData.map((queryPackageGroup, index) => {
    // Get the corresponding package group from the packages prop to get the hidden state
    const packageGroupWithHidden = packages[index]
    const shouldAlwaysIncludeFirstPackage =
      !packageGroupWithHidden ||
      !hasPackageGroupLabel(packageGroupWithHidden) ||
      !!packageGroupWithHidden.baseline

    // Filter out any sub packages that are hidden before
    // summing them into a unified downloads count
    const visiblePackages = queryPackageGroup.packages.filter((p, i) => {
      const hiddenState = packageGroupWithHidden?.packages.find(
        (pg) => pg.name === p.name,
      )?.hidden
      return (i === 0 && shouldAlwaysIncludeFirstPackage) || !hiddenState
    })

    const groupName = packageGroupWithHidden
      ? getPackageGroupLabel(packageGroupWithHidden)
      : queryPackageGroup.packages[0]?.name || 'Unknown'
    const downloadsByDate: Map<number, number> = new Map()

    visiblePackages.forEach((pkg) => {
      pkg.downloads.forEach((d) => {
        // Clamp the data to the floor bin of the start date
        const date = getUtcDay(new Date(d.day))
        if (date < startDate) return

        downloadsByDate.set(
          date.getTime(),
          // Sum the downloads for each date
          (downloadsByDate.get(date.getTime()) || 0) + d.downloads,
        )
      })
    })

    const packageDownloads = visiblePackages.map((pkg) => {
      const packageName = pkg.name || groupName
      const packageDownloadsByDate: Map<number, number> = new Map()

      pkg.downloads.forEach((d) => {
        const date = getUtcDay(new Date(d.day))
        if (date < startDate) return

        packageDownloadsByDate.set(
          date.getTime(),
          (packageDownloadsByDate.get(date.getTime()) || 0) + d.downloads,
        )
      })

      return {
        packageName,
        downloads: Array.from(packageDownloadsByDate.entries()).map(
          ([date, downloads]) => ({
            date: getUtcDay(new Date(date)),
            downloads,
          }),
        ),
      }
    })

    return {
      ...queryPackageGroup,
      label: groupName,
      groupName,
      packageDownloads,
      downloads: Array.from(downloadsByDate.entries()).map(
        ([date, downloads]) => [getUtcDay(new Date(date)), downloads],
      ) as [Date, number][],
    }
  })

  // Prepare data for plotting
  const binnedPackageData = combinedPackageGroups.map((packageGroup) => {
    // Now apply our binning as groupings
    const binned = d3.sort(
      d3.rollup(
        packageGroup.downloads,
        (v) => d3.sum(v, (d) => d[1]),
        (d) => binUnit.floor(d[0]),
      ),
      (d) => d[0],
    )

    const downloads = binned.map((d) => ({
      name: packageGroup.groupName,
      date: getUtcDay(new Date(d[0])),
      downloads: d[1],
    }))

    return {
      ...packageGroup,
      downloads,
      packageDownloads: packageGroup.packageDownloads.map((pkg) => {
        const packageBinned = d3.sort(
          d3.rollup(
            pkg.downloads,
            (v) => d3.sum(v, (d) => d.downloads),
            (d) => binUnit.floor(d.date),
          ),
          (d) => d[0],
        )

        return {
          ...pkg,
          groupName: packageGroup.groupName,
          downloads: packageBinned.map((d) => ({
            groupName: packageGroup.groupName,
            packageName: pkg.packageName,
            date: getUtcDay(new Date(d[0])),
            downloads: d[1],
          })),
        }
      }),
    }
  })

  // Build the baseline divisor.
  //
  // Single-package baseline: divisor[T] = downloads_baseline[T]. Tracked
  // packages divided by this give "% of baseline" — intuitive when there's
  // one familiar reference like react.
  //
  // Multi-package baseline: equal-weighted growth index. Each baseline j is
  // first re-based to its own T0:  ix_j[T] = downloads_j[T] / downloads_j[T0].
  // We average ix across baseline packages to get a unitless multiplier B[T]
  // starting at 1.0 — each member contributes its growth rate equally
  // regardless of size, so the largest member can't dominate the line shape.
  // Tracked packages are then divided by B[T], yielding download values
  // adjusted for baseline growth.
  const baselineGroups = useBaseline
    ? binnedPackageData.filter((_, index) => packages[index]?.baseline)
    : []
  const baselineLineName = getBaselineDisplayName(
    packages.filter((packageGroup) => packageGroup.baseline),
  )
  const isMultiBaseline = baselineGroups.length > 1

  const baselineDivisorByDate = baselineGroups.length
    ? (() => {
        if (!isMultiBaseline) {
          // Single baseline group — divisor is its raw download counts.
          const single = new Map<number, number>()
          baselineGroups[0]?.downloads.forEach((d) => {
            single.set(d.date.getTime(), d.downloads)
          })
          return single.size ? single : undefined
        }
        // Equal-weighted index: index each baseline to its own T0 then average.
        const perPackageIndex = baselineGroups.map((group) => {
          const t0 = group.downloads[0]?.downloads ?? 0
          const ix = new Map<number, number>()
          group.downloads.forEach((d) => {
            ix.set(d.date.getTime(), t0 > 0 ? d.downloads / t0 : 1)
          })
          return ix
        })
        const allDates = new Set<number>()
        perPackageIndex.forEach((ix) =>
          ix.forEach((_, key) => allDates.add(key)),
        )
        const averaged = new Map<number, number>()
        allDates.forEach((key) => {
          let sum = 0
          let count = 0
          perPackageIndex.forEach((ix) => {
            const v = ix.get(key)
            if (v !== undefined) {
              sum += v
              count++
            }
          })
          if (count > 0) averaged.set(key, sum / count)
        })
        return averaged.size ? averaged : undefined
      })()
    : undefined

  const normalizeByBaseline =
    useBaseline && !!baselineDivisorByDate && normalizeBaseline

  const correctedPackageData = binnedPackageData.map((packageGroup) => {
    const first = packageGroup.downloads[0]
    const firstDownloads = first?.downloads ?? 0

    return {
      ...packageGroup,
      downloads: packageGroup.downloads.map((d) => {
        const downloads =
          normalizeByBaseline && baselineDivisorByDate
            ? d.downloads / (baselineDivisorByDate.get(d.date.getTime()) || 1)
            : d.downloads

        return {
          ...d,
          downloads,
          change: downloads - firstDownloads,
        }
      }),
    }
  })

  // Filter out any top-level hidden packages. Baseline series stay in the
  // plot when visible — they render as a flat reference line at 1.0 (every
  // point divided by itself) so users can see the baseline alongside the
  // normalized series.
  const filteredPackageData = correctedPackageData.filter((_, index) => {
    const packageGroupWithHidden = packages[index]
    return packageGroupWithHidden
      ? !isPackageGroupHidden(packageGroupWithHidden)
      : true
  })

  const plotData = filteredPackageData.flatMap((d) =>
    d.downloads.map((download) => ({
      ...download,
      seriesName: download.name,
    })),
  )

  if (
    useBaseline &&
    chartType === 'line' &&
    showBaseline &&
    baselineDivisorByDate
  ) {
    const baselinePoints = [...baselineDivisorByDate.entries()]
      .sort((a, b) => a[0] - b[0])
      .map(([time, divisor]) => ({
        name: baselineLineName,
        seriesName: BASELINE_LINE_SERIES,
        date: getUtcDay(new Date(time)),
        // When normalized, baseline / baseline = 1 at every point.
        // When raw, multi-baseline shows the growth-index multiplier (~1.0+);
        // single-baseline shows the raw divisor value (downloads).
        downloads: normalizeByBaseline ? 1 : divisor,
      }))
    const firstBaselineDownloads = baselinePoints[0]?.downloads ?? 0
    plotData.push(
      ...baselinePoints.map((d) => ({
        ...d,
        change: d.downloads - firstBaselineDownloads,
      })),
    )
  }

  const getStrokeColor = (d: { name?: string; seriesName?: string }) => {
    if (d.seriesName === BASELINE_LINE_SERIES) return BASELINE_LINE_COLOR
    return d.name ? getPackageColor(d.name, packages) : 'currentColor'
  }

  const partialBinEnd = binUnit.floor(now)
  const partialBinStart = binUnit.offset(partialBinEnd, -1)

  // Force complete data mode when using relative change
  const effectiveShowDataMode =
    effectiveTransform === 'normalize-y' ? 'complete' : showDataMode
  const completeTimeData =
    effectiveShowDataMode === 'all'
      ? plotData
      : plotData.filter((d) => d.date < partialBinEnd)
  const timelineScrubberDates =
    viewMode === 'history'
      ? [...new Set(completeTimeData.map((d) => d.date.getTime()))]
          .sort((a, b) => a - b)
          .map((time) => getUtcDay(new Date(time)))
      : []
  const timelineScrubberMaxIndex = timelineScrubberDates.length - 1
  const rawTimelineStartIndex =
    timelineScrubberMaxIndex > 0
      ? getTimelineIndexForTime({
          dates: timelineScrubberDates,
          fallbackIndex: 0,
          time: timelineStart,
        })
      : 0
  const timelineStartIndex =
    timelineScrubberMaxIndex > 0
      ? Math.min(
          clampTimelineIndex(rawTimelineStartIndex, timelineScrubberMaxIndex),
          timelineScrubberMaxIndex - 1,
        )
      : 0
  const rawTimelineEndIndex =
    timelineScrubberMaxIndex > 0
      ? getTimelineIndexForTime({
          dates: timelineScrubberDates,
          fallbackIndex: timelineScrubberMaxIndex,
          time: timelineEnd,
        })
      : timelineScrubberMaxIndex
  const timelineEndIndex =
    timelineScrubberMaxIndex > 0
      ? clampTimelineIndex(
          Math.max(rawTimelineEndIndex, timelineStartIndex + 1),
          timelineScrubberMaxIndex,
        )
      : timelineScrubberMaxIndex
  const hasTimelineZoom =
    timelineScrubberMaxIndex > 0 &&
    (timelineStartIndex > 0 || timelineEndIndex < timelineScrubberMaxIndex)
  const seriesOrder = [
    ...new Set(
      filteredPackageData.map((packageGroup) => packageGroup.groupName),
    ),
  ]
  const plotDataByDate = new Map<
    number,
    Map<string, (typeof plotData)[number]>
  >()

  plotData.forEach((point) => {
    if (!point.seriesName) return

    const dateKey = point.date.getTime()
    const dateMap = plotDataByDate.get(dateKey) ?? new Map()
    dateMap.set(point.seriesName, point)
    plotDataByDate.set(dateKey, dateMap)
  })

  const getStackedRowValue = (
    row: StackedDateRow<(typeof plotData)[number]>,
    seriesName: string,
  ) => {
    const value = row[seriesName]
    return typeof value === 'number' ? value : 0
  }

  const stackRows: Array<StackedDateRow<(typeof plotData)[number]>> = [
    ...new Set(completeTimeData.map((d) => d.date.getTime())),
  ]
    .sort((a, b) => a - b)
    .map((dateKey) => {
      const dateMap = plotDataByDate.get(dateKey)
      const row: StackedDateRow<(typeof plotData)[number]> = {
        date: getUtcDay(new Date(dateKey)),
        pointsBySeries: dateMap ?? new Map(),
      }

      seriesOrder.forEach((seriesName) => {
        row[seriesName] = dateMap?.get(seriesName)?.downloads ?? 0
      })

      return row
    })
  const latestStackRow = stackRows[stackRows.length - 1]
  const stackOrder =
    chartType === 'stacked-stream'
      ? getInsideOutOrderByLatestValue({
          latestValues: seriesOrder.map((seriesName) =>
            latestStackRow ? getStackedRowValue(latestStackRow, seriesName) : 0,
          ),
          sums: seriesOrder.map((seriesName) =>
            d3.sum(stackRows, (row) => getStackedRowValue(row, seriesName)),
          ),
        })
      : d3.range(seriesOrder.length)
  const stackOffset =
    chartType === 'stacked-stream'
      ? d3.stackOffsetWiggle
      : chartType === 'stacked-area'
        ? d3.stackOffsetExpand
        : d3.stackOffsetNone

  const stackedTimeData = d3
    .stack<StackedDateRow<(typeof plotData)[number]>, string>()
    .keys(seriesOrder)
    .value((row, seriesName) => getStackedRowValue(row, seriesName))
    .order(stackOrder)
    .offset(stackOffset)(stackRows)
    .sort((a, b) => a.index - b.index)
    .flatMap((series) =>
      series.map((segment) => {
        const downloads = getStackedRowValue(segment.data, series.key)
        const totalDownloads = seriesOrder.reduce(
          (sum, seriesName) =>
            sum + getStackedRowValue(segment.data, seriesName),
          0,
        )
        const share = totalDownloads > 0 ? downloads / totalDownloads : 0

        return {
          name: series.key,
          seriesName: series.key,
          date: segment.data.date,
          downloads,
          y0: segment[0],
          y1: segment[1],
          label:
            chartType === 'stacked-area'
              ? `${series.key}: ${formatNumber(downloads)} (${percentFormatter.format(share)})`
              : `${series.key}: ${formatNumber(downloads)}`,
        }
      }),
    )

  const latestBarGroups = filteredPackageData
    .map((packageGroup) => ({
      packageGroup,
      name: packageGroup.groupName,
      downloads: d3.sum(packageGroup.downloads, (d) => d.downloads),
    }))
    .filter((group) => group.downloads > 0)
    .sort((a, b) =>
      barSort === 'value'
        ? d3.descending(a.downloads, b.downloads) ||
          a.name.localeCompare(b.name)
        : a.name.localeCompare(b.name),
    )
  const latestBarDomain = latestBarGroups.map((group) => group.name)

  const latestGroupedBarData = latestBarGroups.map((group) => ({
    name: group.name,
    seriesName: group.name,
    downloads: group.downloads,
    label: `${group.name}: ${formatNumber(group.downloads)}`,
  }))

  const latestStackedBarData = latestBarGroups.flatMap((group) => {
    let y0 = 0

    const sortedPackageDownloads = group.packageGroup.packageDownloads
      .map((pkg, packageIndex) => ({
        ...pkg,
        packageIndex,
        totalDownloads: d3.sum(pkg.downloads, (d) => d.downloads),
      }))
      .filter((pkg) => pkg.totalDownloads > 0)
      .sort((a, b) => b.totalDownloads - a.totalDownloads)

    return sortedPackageDownloads.map((pkg) => {
      const point = {
        groupName: group.name,
        packageName: pkg.packageName,
        packageIndex: pkg.packageIndex,
        downloads: pkg.totalDownloads,
        y0,
        y1: y0 + pkg.totalDownloads,
        label: `${pkg.packageName}: ${formatNumber(pkg.totalDownloads)}`,
      }
      y0 += pkg.totalDownloads
      return point
    })
  })
  const getSubPackageColor = (d: { groupName: string; packageIndex: number }) =>
    getSubPackageColorForPackages({ ...d, packages })

  const isLatestGroupedBar = viewMode === 'latest' && chartType === 'bar'
  const isLatestStackedBar =
    viewMode === 'latest' && chartType === 'stacked-bar'
  const isLatestBar = isLatestGroupedBar || isLatestStackedBar
  const isHorizontalBar = isLatestBar && barOrientation === 'horizontal'
  const isLatestVerticalBar = isLatestBar && barOrientation === 'vertical'
  const marginRight = 10
  const marginLeft = isHorizontalBar
    ? getCategoricalAxisLeftMargin({ labels: latestBarDomain, width })
    : 70
  const latestVerticalXAxisLayout = getVerticalCategoricalXAxisLayout({
    height,
    labels: latestBarDomain,
    marginLeft,
    marginRight,
    width,
  })
  const xLabel = isHorizontalBar
    ? 'Downloads'
    : viewMode === 'latest'
      ? null
      : 'Date'
  const yLabel = isHorizontalBar
    ? null
    : chartType === 'stacked-area'
      ? 'Download Share'
      : chartType === 'stacked-stream'
        ? 'Downloads (stream)'
        : effectiveTransform === 'normalize-y'
          ? 'Downloads Growth'
          : normalizeByBaseline
            ? isMultiBaseline
              ? 'Downloads (indexed)'
              : 'Downloads (% of baseline)'
            : 'Downloads'
  const usePercentYAxis =
    chartType === 'stacked-area' ||
    (normalizeByBaseline &&
      !isMultiBaseline &&
      effectiveTransform !== 'normalize-y')
  const yTickFormat = isHorizontalBar
    ? undefined
    : usePercentYAxis
      ? (value: number) => percentFormatter.format(value)
      : formatCompactAxisNumber
  const formatHistoryTooltipValue = usePercentYAxis
    ? (value: number) => percentFormatter.format(value)
    : formatNumber
  const colorLegendEntries = getUniqueColorLegendEntries(
    isLatestStackedBar
      ? latestStackedBarData.map((bar) => ({
          label: `${bar.groupName} / ${bar.packageName}`,
          color: getSubPackageColor(bar),
        }))
      : isLatestGroupedBar
        ? latestGroupedBarData.map((bar) => ({
            label: bar.name,
            color: getPackageColor(bar.name, packages),
          }))
        : plotData
            .map((d) => d.name)
            .filter((name): name is string => name !== undefined)
            .map((name) => ({
              label: name,
              color:
                name === baselineLineName
                  ? BASELINE_LINE_COLOR
                  : getPackageColor(name, packages),
            })),
  )
  const useHistoryAreaFill =
    viewMode === 'history' &&
    (chartType === 'stacked' ||
      chartType === 'stacked-area' ||
      chartType === 'stacked-stream')
  const fillGradients = getChartFillGradients({
    bottomOpacity: isLatestBar ? 0.7 : useHistoryAreaFill ? 0.25 : 0.1,
    entries: colorLegendEntries,
    idPrefix: gradientIdPrefix,
    topOpacity: isLatestBar ? 1 : useHistoryAreaFill ? 0.9 : 0.8,
  })
  const getSeriesGradientFill = (d: { name?: string; seriesName?: string }) =>
    getChartFillGradientUrl({
      color: getStrokeColor(d),
      gradients: fillGradients,
    })
  const getSubPackageGradientFill = (d: {
    groupName: string
    packageIndex: number
  }) =>
    getChartFillGradientUrl({
      color: getSubPackageColor(d),
      gradients: fillGradients,
    })

  const historyComplete: Array<NpmHistoryPoint> = plotData
    .filter((datum) => datum.date < partialBinEnd)
    .map((datum) => {
      const value =
        effectiveTransform === 'normalize-y' ? datum.change : datum.downloads
      return {
        color: getStrokeColor(datum),
        date: datum.date,
        id: `${datum.seriesName}:${datum.date.getTime()}:complete`,
        label: `${datum.name}: ${formatHistoryTooltipValue(value)}`,
        seriesName: datum.seriesName,
        seriesLabel: datum.name,
        value,
      }
    })
  const historyPartial: Array<NpmHistoryPoint> =
    effectiveShowDataMode === 'all'
      ? plotData
          .filter((datum) => datum.date >= partialBinStart)
          .map((datum) => {
            const value =
              effectiveTransform === 'normalize-y'
                ? datum.change
                : datum.downloads
            return {
              color: getStrokeColor(datum),
              date: datum.date,
              id: `${datum.seriesName}:${datum.date.getTime()}:partial`,
              label: `${datum.name}: ${formatHistoryTooltipValue(value)}`,
              seriesName: datum.seriesName,
              seriesLabel: datum.name,
              value,
            }
          })
      : []
  const stackedHistory: Array<NpmStackedHistoryPoint> = stackedTimeData.map(
    (datum) => ({
      color: getStrokeColor(datum),
      date: datum.date,
      fill: getSeriesGradientFill(datum),
      id: `${datum.seriesName}:${datum.date.getTime()}`,
      label: datum.label,
      seriesName: datum.seriesName,
      value: datum.y1,
      value1: datum.y0,
      value2: datum.y1,
    }),
  )
  const bars: Array<NpmBarPoint> = isLatestStackedBar
    ? latestStackedBarData.map((datum) => ({
        category: datum.groupName,
        color: getSubPackageColor(datum),
        fill: getSubPackageGradientFill(datum),
        id: `${datum.groupName}:${datum.packageName}:${datum.packageIndex}`,
        label: datum.label,
        seriesName: datum.packageName,
        value1: datum.y0,
        value2: datum.y1,
      }))
    : latestGroupedBarData.map((datum) => ({
        category: datum.name,
        color: getPackageColor(datum.name, packages),
        fill: getSeriesGradientFill(datum),
        id: datum.name,
        label: datum.label,
        seriesName: datum.seriesName,
        value1: 0,
        value2: datum.downloads,
      }))
  const timelineDomainStart = hasTimelineZoom
    ? timelineScrubberDates[timelineStartIndex]
    : undefined
  const timelineDomainEnd = hasTimelineZoom
    ? timelineScrubberDates[timelineEndIndex]
    : undefined
  const chartTimelineDomain: [Date, Date] | undefined =
    timelineDomainStart && timelineDomainEnd
      ? [timelineDomainStart, timelineDomainEnd]
      : undefined

  return (
    <div ref={containerRef} className="w-full">
      {chartActions}
      {width > 0 ? (
        <ChartFigure
          colorLegendEntries={colorLegendEntries}
          definitionRevision={chartDefinitionRevision}
          footer={
            viewMode === 'history' && timelineScrubberMaxIndex > 0 ? (
              <TimelineRangeScrubber
                dates={timelineScrubberDates}
                endIndex={timelineEndIndex}
                marginLeft={marginLeft}
                marginRight={marginRight}
                onRangeChange={(nextStartIndex, nextEndIndex) => {
                  if (
                    nextStartIndex <= 0 &&
                    nextEndIndex >= timelineScrubberMaxIndex
                  ) {
                    onTimelineRangeChange?.({
                      end: undefined,
                      start: undefined,
                    })
                    return
                  }

                  onTimelineRangeChange?.({
                    end: timelineScrubberDates[nextEndIndex]?.getTime(),
                    start: timelineScrubberDates[nextStartIndex]?.getTime(),
                  })
                }}
                startIndex={timelineStartIndex}
              />
            ) : null
          }
          height={height}
          input={{
            barOrientation,
            bars,
            chartType,
            gradients: getTanStackGradients(fillGradients),
            historyComplete,
            historyPartial,
            marginBottom: isLatestVerticalBar
              ? latestVerticalXAxisLayout.marginBottom
              : 70,
            marginLeft,
            marginRight,
            stackedHistory,
            timelineDomain: chartTimelineDomain,
            viewMode,
            xLabel: xLabel ?? undefined,
            yFormat: yTickFormat,
            yLabel: yLabel ?? undefined,
          }}
          legendRef={legendRef}
          onRenderedChange={handleChartRenderedChange}
          showLegend={showLegend}
          svgRef={svgRef}
          width={width}
        />
      ) : (
        <div style={{ height }} />
      )}
    </div>
  )
}
````

## File: src/components/npm-stats/NPMSummary.tsx
````typescript
import * as React from 'react'
import { Suspense } from 'react'
import { useSuspenseQuery } from '@tanstack/react-query'
import { BlankErrorBoundary } from '~/components/BlankErrorBoundary'
import { ossStatsQuery, recentDownloadsQuery } from '~/queries/stats'
import { useNpmDownloadCounter } from '~/hooks/useNpmDownloadCounter'
import type { Library } from '~/libraries'

/**
 * Format a number with appropriate suffix (K, M, B)
 */
function formatNumber(num: number): string {
  if (num >= 1_000_000_000) {
    return `${(num / 1_000_000_000).toFixed(1)}B`
  }
  if (num >= 1_000_000) {
    return `${(num / 1_000_000).toFixed(1)}M`
  }
  if (num >= 1_000) {
    return `${(num / 1_000).toFixed(1)}K`
  }
  return num.toLocaleString()
}

/**
 * Animated counter component for all-time downloads
 */
function AnimatedDownloadCounter({
  npmData,
}: {
  npmData: {
    totalDownloads: number
    ratePerDay?: number
    updatedAt?: number
  }
}) {
  const ref = useNpmDownloadCounter(npmData)
  const initialCount = npmData.totalDownloads ?? 0
  return (
    <span ref={ref} style={{ fontVariantNumeric: 'tabular-nums' }}>
      {initialCount.toLocaleString()}
    </span>
  )
}

/**
 * Stat card component
 */
function StatCard({
  value,
  label,
  animated,
  npmData,
}: {
  value?: number
  label: string
  animated?: boolean
  npmData?: {
    totalDownloads: number
    ratePerDay?: number
    updatedAt?: number
  }
}) {
  return (
    <div className="flex flex-col items-center text-center">
      <div
        className="text-xl sm:text-xl md:text-2xl font-bold text-gray-900 dark:text-white"
        style={{ fontVariantNumeric: 'tabular-nums' }}
      >
        {animated && npmData ? (
          <AnimatedDownloadCounter npmData={npmData} />
        ) : (
          formatNumber(value ?? 0)
        )}
      </div>
      <div className="text-xs sm:text-sm text-gray-500 dark:text-gray-400 mt-1">
        {label}
      </div>
    </div>
  )
}

/**
 * Content component that fetches and displays NPM stats
 */
function NPMSummaryContent({ library }: { library: Library }) {
  // Fetch all-time stats (includes ratePerDay for animation)
  const { data: ossStats } = useSuspenseQuery(ossStatsQuery({ library }))

  // Fetch recent download stats (daily, weekly, monthly)
  const { data: recentStats } = useSuspenseQuery(
    recentDownloadsQuery({ library }),
  )

  return (
    <div className="my-6">
      <p className="text-sm text-gray-500 dark:text-gray-400 mb-4">
        View download statistics for TanStack {library.name} packages. Compare
        different time periods and track usage trends.
      </p>

      <p className="text-xs text-gray-400 dark:text-gray-500 my-3 italic">
        *These top summary stats account for core packages, legacy package
        names, and all framework adapters.
      </p>

      <div className="grid grid-cols-2 gap-4 sm:gap-6 md:grid-cols-2 xl:grid-cols-4 py-6">
        <StatCard label="All Time Downloads" animated npmData={ossStats.npm} />
        <StatCard
          value={recentStats.monthlyDownloads}
          label="Monthly Downloads"
        />
        <StatCard
          value={recentStats.weeklyDownloads}
          label="Weekly Downloads"
        />
        <StatCard value={recentStats.dailyDownloads} label="Daily Downloads" />
      </div>
    </div>
  )
}

/**
 * Skeleton loading state
 */
function NPMSummarySkeleton() {
  return (
    <div className="mb-6">
      <div className="h-7 w-64 bg-gray-200 dark:bg-gray-700 rounded animate-pulse mb-2" />
      <div className="h-5 w-96 max-w-full bg-gray-200 dark:bg-gray-700 rounded animate-pulse mb-4" />

      <div className="grid grid-cols-2 md:grid-cols-4 gap-4 sm:gap-6">
        {[...Array(4)].map((_, i) => (
          <div key={i} className="flex flex-col items-center">
            <div className="h-9 w-24 bg-gray-200 dark:bg-gray-700 rounded animate-pulse" />
            <div className="h-4 w-20 bg-gray-200 dark:bg-gray-700 rounded animate-pulse mt-2" />
          </div>
        ))}
      </div>
    </div>
  )
}

/**
 * NPM Summary component showing all-time, monthly, weekly, and daily downloads
 * Uses Suspense for loading state and ErrorBoundary for error handling
 */
export function NPMSummary({ library }: { library: Library }) {
  return (
    <Suspense fallback={<NPMSummarySkeleton />}>
      <BlankErrorBoundary>
        <NPMSummaryContent library={library} />
      </BlankErrorBoundary>
    </Suspense>
  )
}
````

## File: src/components/npm-stats/PackagePills.tsx
````typescript
import * as React from 'react'
import { twMerge } from 'tailwind-merge'
import {
  XIcon,
  PlusIcon,
  EyeIcon,
  EyeSlashIcon,
  DotsThreeVerticalIcon,
} from '@phosphor-icons/react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@radix-ui/react-dropdown-menu'
import { Tooltip } from '~/components/Tooltip'
import {
  type PackageGroup,
  getPackageColor,
  getPackageGroupLabel,
  hasPackageGroupLabel,
  isPackageGroupHidden,
} from './shared'

export type PackagePillProps = {
  packageGroup: PackageGroup
  index: number
  allPackageGroups: PackageGroup[]
  error?: string
  onColorClick: (packageName: string, event: React.MouseEvent) => void
  onToggleVisibility: (index: number, packageName: string) => void
  onRemove: (index: number) => void
  // Optional advanced features (main page only)
  onCombinePackage?: (packageName: string) => void
  onLabelChange?: (index: number, label: string) => void
  onRemoveFromGroup?: (mainPackage: string, subPackage: string) => void
  openMenuPackage?: string | null
  onMenuOpenChange?: (packageName: string, open: boolean) => void
}

export function PackagePill({
  packageGroup,
  index,
  allPackageGroups,
  error,
  onColorClick,
  onToggleVisibility,
  onRemove,
  onCombinePackage,
  onLabelChange,
  onRemoveFromGroup,
  openMenuPackage,
  onMenuOpenChange,
}: PackagePillProps) {
  const currentLabel = packageGroup.label ?? ''
  const [draftLabel, setDraftLabel] = React.useState(currentLabel)
  const labelInputId = React.useId()

  React.useEffect(() => {
    setDraftLabel(currentLabel)
  }, [currentLabel])

  const mainPackage = packageGroup.packages[0]
  if (!mainPackage) return null

  const hasLabel = hasPackageGroupLabel(packageGroup)
  const packageList = packageGroup.packages
  const listedPackages = hasLabel
    ? packageList
    : packageList.filter((p) => p.name !== mainPackage.name)
  const showPackageList = listedPackages.length > 0
  const showPackageCount = !hasLabel && showPackageList
  const color = getPackageColor(mainPackage.name, allPackageGroups)
  const label = getPackageGroupLabel(packageGroup)
  const isGroupHidden = isPackageGroupHidden(packageGroup)

  const showAdvancedMenu =
    onMenuOpenChange && (onCombinePackage || onLabelChange)
  const handleLabelBlur = () => {
    if (draftLabel !== currentLabel) {
      onLabelChange?.(index, draftLabel)
    }
  }

  return (
    <div
      className="flex flex-col items-start rounded text-xs leading-none text-gray-900 dark:text-gray-100"
      style={{
        backgroundColor: `${color}20`,
      }}
    >
      <div className="flex w-full items-stretch">
        <Tooltip content="Change color">
          <button
            onClick={(e) => onColorClick(mainPackage.name, e)}
            className="flex w-5 shrink-0 hover:opacity-80"
          >
            <div
              className="w-full rounded-l"
              style={{ backgroundColor: color }}
            />
          </button>
        </Tooltip>
        <div className="flex min-w-0 flex-1 items-center gap-0.5 py-1 pl-1.5 pr-2">
          <Tooltip content="Toggle package visibility">
            <button
              onClick={() =>
                onToggleVisibility(index, hasLabel ? label : mainPackage.name)
              }
              className={twMerge(
                'hover:text-blue-500 flex min-w-0 max-w-48 items-center gap-0.5',
                isGroupHidden ? 'opacity-50' : '',
              )}
            >
              <span className="truncate">{label}</span>
              {isGroupHidden ? (
                <EyeSlashIcon className="size-3.5 shrink-0" />
              ) : null}
            </button>
          </Tooltip>
          {showPackageCount ? (
            <span className="rounded-sm border border-current px-1 text-[11px] font-black leading-none text-black/70 opacity-80 dark:text-white/70">
              + {listedPackages.length}
            </span>
          ) : null}

          {/* Advanced dropdown menu (main page only) */}
          {showAdvancedMenu && (
            <div className="relative flex items-center">
              <DropdownMenu
                open={openMenuPackage === mainPackage.name}
                onOpenChange={(open) =>
                  onMenuOpenChange(mainPackage.name, open)
                }
              >
                <Tooltip content="More options">
                  <DropdownMenuTrigger asChild>
                    <button className="px-0.5 hover:text-blue-500">
                      <DotsThreeVerticalIcon className="size-3.5" />
                    </button>
                  </DropdownMenuTrigger>
                </Tooltip>
                <DropdownMenuContent
                  className="z-50 max-h-[var(--radix-dropdown-menu-content-available-height)] min-w-[200px] overflow-y-auto rounded-lg bg-white p-2 shadow-lg dark:bg-gray-800"
                  collisionPadding={8}
                  sideOffset={5}
                >
                  <div className="flex justify-between items-center mb-2">
                    <span className="text-sm font-medium">Options</span>
                  </div>
                  <div className="space-y-1">
                    {onLabelChange && (
                      <div className="px-2 py-1.5 space-y-1">
                        <label
                          className="block text-xs font-medium text-gray-500"
                          htmlFor={labelInputId}
                        >
                          Label
                        </label>
                        <input
                          className="w-full rounded border border-gray-500/20 bg-white px-2 py-1 text-sm outline-none focus:border-blue-500 dark:bg-gray-900"
                          id={labelInputId}
                          maxLength={80}
                          onBlur={handleLabelBlur}
                          onChange={(event) =>
                            setDraftLabel(event.currentTarget.value)
                          }
                          onClick={(event) => event.stopPropagation()}
                          onKeyDown={(event) => {
                            event.stopPropagation()
                            if (event.key === 'Enter') {
                              event.currentTarget.blur()
                            }
                          }}
                          placeholder={mainPackage.name}
                          value={draftLabel}
                        />
                      </div>
                    )}
                    <DropdownMenuItem
                      onSelect={(e) => {
                        e.preventDefault()
                        onToggleVisibility(
                          index,
                          hasLabel ? label : mainPackage.name,
                        )
                      }}
                      className="w-full px-2 py-1.5 text-left text-sm rounded hover:bg-gray-500/20 flex items-center gap-2 outline-none cursor-pointer"
                    >
                      {isGroupHidden ? (
                        <EyeSlashIcon className="text-sm" />
                      ) : (
                        <EyeIcon className="text-sm" />
                      )}
                      {isGroupHidden ? 'Show Package' : 'Hide Package'}
                    </DropdownMenuItem>
                    <DropdownMenuItem
                      onSelect={(e) => {
                        e.preventDefault()
                        onColorClick(
                          mainPackage.name,
                          e as unknown as React.MouseEvent,
                        )
                      }}
                      className="w-full px-2 py-1.5 text-left text-sm rounded hover:bg-gray-500/20 flex items-center gap-2 outline-none cursor-pointer"
                    >
                      <div
                        className="w-3 h-3 rounded-full"
                        style={{ backgroundColor: color }}
                      />
                      Change Color
                    </DropdownMenuItem>
                    {showPackageList && onRemoveFromGroup && (
                      <>
                        <div className="h-px bg-gray-500/20 my-1" />
                        <div className="px-2 py-1 text-xs font-medium text-gray-500">
                          {hasLabel ? 'Packages' : 'Sub-packages'}
                        </div>
                        {listedPackages.map((subPackage) => (
                          <DropdownMenuItem
                            key={subPackage.name}
                            onSelect={(e) => {
                              e.preventDefault()
                              onToggleVisibility(index, subPackage.name)
                            }}
                            className="w-full px-2 py-1.5 text-left text-sm rounded hover:bg-gray-500/20 flex items-center gap-2 outline-none cursor-pointer"
                          >
                            <div className="flex-1 flex items-center justify-between">
                              <div className="flex items-center gap-2">
                                {subPackage.hidden ? (
                                  <EyeSlashIcon className="text-sm" />
                                ) : (
                                  <EyeIcon className="text-sm" />
                                )}
                                <span
                                  className={
                                    subPackage.hidden ? 'opacity-50' : ''
                                  }
                                >
                                  {subPackage.name}
                                </span>
                              </div>
                              <button
                                onClick={(e) => {
                                  e.stopPropagation()
                                  onRemoveFromGroup(
                                    mainPackage.name,
                                    subPackage.name,
                                  )
                                }}
                                className="p-1 text-gray-400 hover:text-red-500"
                              >
                                <XIcon className="w-3 h-3" />
                              </button>
                            </div>
                          </DropdownMenuItem>
                        ))}
                      </>
                    )}
                    {onCombinePackage && (
                      <DropdownMenuItem
                        onSelect={(e) => {
                          e.preventDefault()
                          onCombinePackage(mainPackage.name)
                        }}
                        className="w-full px-2 py-1.5 text-left text-sm rounded hover:bg-gray-500/20 flex items-center gap-2 outline-none cursor-pointer"
                      >
                        <PlusIcon className="text-sm" />
                        Add Packages
                      </DropdownMenuItem>
                    )}
                  </div>
                </DropdownMenuContent>
              </DropdownMenu>
            </div>
          )}

          <button
            onClick={() => onRemove(index)}
            className="ml-auto pl-0.5 text-gray-500 hover:text-red-500"
          >
            <XIcon className="size-3.5" />
          </button>
        </div>
      </div>
      {error && (
        <div className="mt-0.5 rounded bg-red-500/10 px-1 py-0.5 font-mono text-[10px] font-medium leading-none text-red-500">
          {error}
        </div>
      )}
    </div>
  )
}

export type PackagePillsProps = {
  packageGroups: PackageGroup[]
  queryData?: Array<{ error?: string } | undefined>
  onColorClick: (packageName: string, event: React.MouseEvent) => void
  onToggleVisibility: (index: number, packageName: string) => void
  onRemove: (index: number) => void
  // Optional advanced features (main page only)
  onCombinePackage?: (packageName: string) => void
  onLabelChange?: (index: number, label: string) => void
  onRemoveFromGroup?: (mainPackage: string, subPackage: string) => void
  openMenuPackage?: string | null
  onMenuOpenChange?: (packageName: string, open: boolean) => void
}

export function PackagePills({
  packageGroups,
  queryData,
  onColorClick,
  onToggleVisibility,
  onRemove,
  onCombinePackage,
  onLabelChange,
  onRemoveFromGroup,
  openMenuPackage,
  onMenuOpenChange,
}: PackagePillsProps) {
  return (
    <div className="flex flex-wrap gap-1">
      {packageGroups.map((pkg, index) => {
        if (pkg.baseline) return null
        return (
          <PackagePill
            key={pkg.packages[0]?.name ?? index}
            packageGroup={pkg}
            index={index}
            allPackageGroups={packageGroups}
            error={queryData?.[index]?.error}
            onColorClick={onColorClick}
            onToggleVisibility={onToggleVisibility}
            onRemove={onRemove}
            onCombinePackage={onCombinePackage}
            onLabelChange={onLabelChange}
            onRemoveFromGroup={onRemoveFromGroup}
            openMenuPackage={openMenuPackage}
            onMenuOpenChange={onMenuOpenChange}
          />
        )
      })}
    </div>
  )
}
````

## File: src/components/npm-stats/PackageSearch.tsx
````typescript
import * as React from 'react'
import { useDebouncedValue } from '@tanstack/react-pacer'
import { MagnifyingGlassIcon } from '@phosphor-icons/react'
import { keepPreviousData, useQuery } from '@tanstack/react-query'
import { Command } from 'cmdk'
import { twMerge } from 'tailwind-merge'
import { Spinner } from '~/components/Spinner'

type NpmSearchResult = {
  name: string
  description?: string
  version?: string
  publisher?: { username?: string }
}

const CREATE_ITEM_VALUE = '__create__'
const MIN_SEARCH_QUERY_LENGTH = 2

export type PackageSearchProps = {
  onSelect: (packageName: string) => void
  placeholder?: string
  autoFocus?: boolean
}

export function PackageSearch({
  onSelect,
  placeholder = 'Search for a package...',
  autoFocus = false,
}: PackageSearchProps) {
  const [inputValue, setInputValue] = React.useState('')
  const [open, setOpen] = React.useState(false)
  const containerRef = React.useRef<HTMLDivElement>(null)

  const [debouncedInputValue] = useDebouncedValue(inputValue, {
    wait: 150,
  })

  React.useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (
        containerRef.current &&
        !containerRef.current.contains(event.target as Node)
      ) {
        setOpen(false)
      }
    }

    document.addEventListener('mousedown', handleClickOutside)
    return () => {
      document.removeEventListener('mousedown', handleClickOutside)
    }
  }, [])

  const trimmedInput = debouncedInputValue.trim()
  const hasUsableQuery = trimmedInput.length >= MIN_SEARCH_QUERY_LENGTH

  const searchQuery = useQuery({
    queryKey: ['npm-search', trimmedInput],
    queryFn: async () => {
      const response = await fetch(
        `https://registry.npmjs.org/-/v1/search?text=${encodeURIComponent(
          trimmedInput,
        )}&size=10`,
      )
      const data = (await response.json()) as {
        objects: Array<{ package: NpmSearchResult }>
      }
      return data.objects.map((r) => r.package)
    },
    enabled: hasUsableQuery,
    placeholderData: keepPreviousData,
  })

  const searchResults = hasUsableQuery ? (searchQuery.data ?? []) : []
  const showCreateItem =
    hasUsableQuery &&
    trimmedInput.length > 0 &&
    !searchResults.some((d) => d.name === trimmedInput)

  const handleSelect = (value: string) => {
    if (value === CREATE_ITEM_VALUE) {
      if (!trimmedInput) return
      onSelect(trimmedInput)
    } else {
      const match = searchResults.find((item) => item.name === value)
      if (!match) return
      onSelect(match.name)
    }
    setInputValue('')
    setOpen(false)
  }

  const showList = open && inputValue.length > 0

  return (
    <div className="relative w-full max-w-[250px]" ref={containerRef}>
      <Command
        className="w-full"
        shouldFilter={false}
        loop
        label="Search npm packages"
      >
        <div className="relative text-xs">
          <MagnifyingGlassIcon className="pointer-events-none absolute left-1.5 top-1/2 size-3.5 -translate-y-1/2 text-gray-500" />
          <Command.Input
            placeholder={placeholder}
            className="h-6 w-full min-w-[180px] rounded bg-gray-500/10 py-0.5 pl-6 pr-6 text-xs outline-none"
            value={inputValue}
            onValueChange={setInputValue}
            onFocus={() => setOpen(true)}
            // eslint-disable-next-line jsx-a11y/no-autofocus
            autoFocus={autoFocus}
          />
          {searchQuery.isFetching && (
            <div className="pointer-events-none absolute bottom-0 right-1.5 top-0 flex items-center justify-center">
              <Spinner className="text-xs" />
            </div>
          )}
        </div>
        <Command.List
          className={twMerge(
            'absolute z-10 mt-1 max-h-56 w-full overflow-auto rounded bg-white text-xs shadow-lg divide-y divide-gray-500/10 dark:bg-gray-800',
            !showList && 'hidden',
          )}
        >
          {inputValue.trim().length < MIN_SEARCH_QUERY_LENGTH ? (
            <div className="px-2 py-1.5 text-gray-500 dark:text-gray-400">
              Keep typing to search...
            </div>
          ) : searchQuery.isLoading ? (
            <div className="px-2 py-1.5 text-gray-500 dark:text-gray-400">
              Searching...
            </div>
          ) : !searchResults.length && !showCreateItem ? (
            <div className="px-2 py-1.5 text-gray-500 dark:text-gray-400">
              No packages found
            </div>
          ) : null}
          {showCreateItem && (
            <Command.Item
              key={CREATE_ITEM_VALUE}
              value={CREATE_ITEM_VALUE}
              onSelect={handleSelect}
              className="cursor-pointer px-2 py-1.5 hover:bg-gray-500/20 data-[selected=true]:bg-gray-500/20"
            >
              <div className="font-medium">Use "{trimmedInput}"</div>
            </Command.Item>
          )}
          {searchResults.map((item) => (
            <Command.Item
              key={item.name}
              value={item.name}
              onSelect={handleSelect}
              className="cursor-pointer px-2 py-1.5 hover:bg-gray-500/20 data-[selected=true]:bg-gray-500/20"
            >
              <div className="font-medium">{item.name}</div>
              {item.description ? (
                <div className="text-[11px] leading-snug text-gray-500 dark:text-gray-400">
                  {item.description}
                </div>
              ) : null}
              <div className="text-[10px] leading-snug text-gray-400 dark:text-gray-500">
                {item.version ? `v${item.version}` : ''}
                {item.version && item.publisher?.username ? ' • ' : ''}
                {item.publisher?.username}
              </div>
            </Command.Item>
          ))}
        </Command.List>
      </Command>
    </div>
  )
}
````

## File: src/components/npm-stats/PopularComparisons.tsx
````typescript
import * as React from 'react'
import { Link } from '@tanstack/react-router'
import { getPackageGroupLabel, type PackageGroup } from './shared'
import { defaultColors } from '~/utils/npm-packages'

export type ComparisonGroup = {
  title: string
  packageGroups: PackageGroup[]
}

type PopularComparisonsProps = {
  comparisons: ComparisonGroup[]
  // Optional: customize the link target
  linkTo?: string
  // Optional: callback when a comparison is clicked
  onSelect?: (comparison: ComparisonGroup) => void
}

const PREVIEW_PACKAGE_LIMIT = 8

export function PopularComparisons({
  comparisons,
  linkTo = '.',
  onSelect,
}: PopularComparisonsProps) {
  return (
    <div>
      <h2 className="text-lg font-semibold mb-3">Presets</h2>
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-3">
        {comparisons.map((comparison) => {
          const baselinePackages = comparison.packageGroups.filter(
            (pg) => pg.baseline,
          )
          const previewPackageGroups = comparison.packageGroups
            .filter((d) => !d.baseline)
            .slice(0, PREVIEW_PACKAGE_LIMIT)
          const hiddenPreviewCount =
            comparison.packageGroups.filter((d) => !d.baseline).length -
            previewPackageGroups.length

          const content = (
            <div className="space-y-2">
              <div className="space-y-1.5">
                <div>
                  <h3 className="text-sm font-medium leading-tight">
                    {comparison.title}
                  </h3>
                </div>
                <div className="flex flex-wrap gap-x-2 gap-y-1">
                  {previewPackageGroups.map((packageGroup) => (
                    <div
                      key={getPackageGroupLabel(packageGroup)}
                      className="flex items-center gap-1 text-xs leading-tight"
                    >
                      <div
                        className="w-2.5 h-2.5 rounded-full"
                        style={{
                          backgroundColor:
                            packageGroup.color || defaultColors[0],
                        }}
                      />
                      <span>{getPackageGroupLabel(packageGroup)}</span>
                    </div>
                  ))}
                  {hiddenPreviewCount > 0 && (
                    <div className="text-xs leading-tight text-gray-500 dark:text-gray-400">
                      + {hiddenPreviewCount} more
                    </div>
                  )}
                </div>
              </div>
              {baselinePackages.length > 0 && (
                <div className="flex items-center gap-1 text-xs flex-wrap">
                  <div className="font-medium">Baseline:</div>
                  {baselinePackages.map((pg, i) => (
                    <React.Fragment key={getPackageGroupLabel(pg) || i}>
                      {i > 0 && (
                        <span className="text-blue-500/60 text-xs">+</span>
                      )}
                      <div className="bg-blue-500/10 text-blue-700 dark:text-blue-300 rounded px-1.5 py-0.5 leading-none font-bold">
                        {getPackageGroupLabel(pg)}
                      </div>
                    </React.Fragment>
                  ))}
                </div>
              )}
            </div>
          )

          if (onSelect) {
            return (
              <button
                key={comparison.title}
                onClick={() => onSelect(comparison)}
                className="block p-3 bg-gray-500/10 hover:bg-gray-500/20 rounded-md transition-colors text-left w-full"
              >
                {content}
              </button>
            )
          }

          return (
            <Link
              key={comparison.title}
              to={linkTo}
              search={(prev: Record<string, unknown>) => ({
                ...prev,
                packageGroups: comparison.packageGroups,
                bucketOffset: 0,
              })}
              resetScroll={false}
              onClick={() => {
                window.scrollTo({
                  top: 0,
                  behavior: 'smooth',
                })
              }}
              className="block p-3 bg-gray-500/10 hover:bg-gray-500/20 rounded-md transition-colors"
            >
              {content}
            </Link>
          )
        })}
      </div>
    </div>
  )
}
````

## File: src/components/npm-stats/Resizable.tsx
````typescript
import * as React from 'react'

export type ResizeChangeOptions = {
  replace: boolean
}

export type ResizableSizeChange = {
  height?: number
  width?: number | undefined
}

export type ResizableProps = {
  height: number
  width?: number
  onSizeChange: (
    size: ResizableSizeChange,
    options: ResizeChangeOptions,
  ) => void
  children: React.ReactNode
  minHeight?: number
  minWidth?: number
  fullWidthSnapThreshold?: number
  enableWidthResize?: boolean
}

type ResizePreviewSize = {
  height: number
  width: number
}

const dimensionTooltipStyles =
  'pointer-events-none absolute z-50 whitespace-nowrap rounded bg-gray-900 px-1.5 py-0.5 text-[10px] font-medium leading-4 text-white shadow-sm dark:bg-gray-100 dark:text-gray-900'

export function Resizable({
  height,
  width,
  onSizeChange,
  children,
  minHeight = 300,
  minWidth = 320,
  fullWidthSnapThreshold = 12,
  enableWidthResize = true,
}: ResizableProps) {
  const [isHeightDragging, setIsHeightDragging] = React.useState(false)
  const [isWidthDragging, setIsWidthDragging] = React.useState(false)
  const [isCornerDragging, setIsCornerDragging] = React.useState(false)
  const [containerWidth, setContainerWidth] = React.useState(0)
  const [previewSize, setPreviewSize] =
    React.useState<ResizePreviewSize | null>(null)
  const containerRef = React.useRef<HTMLDivElement>(null)
  const startYRef = React.useRef<number>(0)
  const startXRef = React.useRef<number>(0)
  const startHeightRef = React.useRef<number>(height)
  const startWidthRef = React.useRef<number>(width ?? 0)

  const onSizeChangeRef = React.useRef(onSizeChange)
  React.useEffect(() => {
    onSizeChangeRef.current = onSizeChange
  })

  React.useEffect(() => {
    const container = containerRef.current
    if (!container) return

    const updateContainerWidth = () => {
      setContainerWidth(Math.round(container.getBoundingClientRect().width))
    }

    updateContainerWidth()

    const ownerWindow = container.ownerDocument.defaultView
    if (!ownerWindow?.ResizeObserver) {
      ownerWindow?.addEventListener('resize', updateContainerWidth)
      return () => {
        ownerWindow?.removeEventListener('resize', updateContainerWidth)
      }
    }

    const resizeObserver = new ownerWindow.ResizeObserver((entries) => {
      const entry = entries[0]
      if (!entry) return
      setContainerWidth(Math.round(entry.contentRect.width))
    })

    resizeObserver.observe(container)
    return () => {
      resizeObserver.disconnect()
    }
  }, [])

  const getMaxWidth = React.useCallback(() => {
    return (
      containerWidth ||
      Math.round(containerRef.current?.getBoundingClientRect().width ?? 0)
    )
  }, [containerWidth])

  const getHeightFromDelta = React.useCallback(
    (deltaY: number) => Math.max(minHeight, startHeightRef.current + deltaY),
    [minHeight],
  )

  const getWidthFromDelta = React.useCallback(
    ({ deltaX, maxWidth }: { deltaX: number; maxWidth: number }) => {
      const rawWidth = startWidthRef.current + deltaX * 2
      const nextMinWidth = Math.min(minWidth, maxWidth)
      const nextWidth = Math.max(
        nextMinWidth,
        Math.min(maxWidth, Math.round(rawWidth)),
      )

      if (nextWidth >= maxWidth - fullWidthSnapThreshold) {
        return undefined
      }

      return nextWidth
    },
    [fullWidthSnapThreshold, minWidth],
  )

  const handleHeightMouseDown = React.useCallback(
    (e: React.MouseEvent<HTMLDivElement>) => {
      e.preventDefault()
      setIsHeightDragging(true)
      const currentWidth = width ?? getMaxWidth()
      setPreviewSize({ height, width: currentWidth })
      startYRef.current = e.clientY
      startHeightRef.current = height

      const ownerDocument = e.currentTarget.ownerDocument
      const previousCursor = ownerDocument.body.style.cursor
      const previousUserSelect = ownerDocument.body.style.userSelect
      ownerDocument.body.style.cursor = 'ns-resize'
      ownerDocument.body.style.userSelect = 'none'

      const handleMouseMove = (e: MouseEvent) => {
        const deltaY = e.clientY - startYRef.current
        const newHeight = getHeightFromDelta(deltaY)
        setPreviewSize({ height: newHeight, width: currentWidth })
        onSizeChangeRef.current({ height: newHeight }, { replace: true })
      }

      const handleMouseUp = (e: MouseEvent) => {
        const deltaY = e.clientY - startYRef.current
        const newHeight = getHeightFromDelta(deltaY)
        if (newHeight !== startHeightRef.current) {
          onSizeChangeRef.current({ height: newHeight }, { replace: false })
        } else {
          onSizeChangeRef.current({}, { replace: false })
        }

        setIsHeightDragging(false)
        setPreviewSize(null)
        ownerDocument.body.style.cursor = previousCursor
        ownerDocument.body.style.userSelect = previousUserSelect
        ownerDocument.removeEventListener('mousemove', handleMouseMove)
        ownerDocument.removeEventListener('mouseup', handleMouseUp)
      }

      ownerDocument.addEventListener('mousemove', handleMouseMove)
      ownerDocument.addEventListener('mouseup', handleMouseUp)
    },
    [getHeightFromDelta, getMaxWidth, height, width],
  )

  const handleWidthMouseDown = React.useCallback(
    (e: React.MouseEvent<HTMLDivElement>) => {
      const maxWidth = getMaxWidth()
      if (!maxWidth) return

      e.preventDefault()
      setIsWidthDragging(true)
      startXRef.current = e.clientX
      startWidthRef.current = Math.min(width ?? maxWidth, maxWidth)
      setPreviewSize({ height, width: startWidthRef.current })

      const ownerDocument = e.currentTarget.ownerDocument
      const previousCursor = ownerDocument.body.style.cursor
      const previousUserSelect = ownerDocument.body.style.userSelect
      ownerDocument.body.style.cursor = 'ew-resize'
      ownerDocument.body.style.userSelect = 'none'

      const handleMouseMove = (e: MouseEvent) => {
        const deltaX = e.clientX - startXRef.current
        const newWidth = getWidthFromDelta({ deltaX, maxWidth })
        setPreviewSize({ height, width: newWidth ?? maxWidth })
        onSizeChangeRef.current({ width: newWidth }, { replace: true })
      }

      const handleMouseUp = (e: MouseEvent) => {
        const deltaX = e.clientX - startXRef.current
        const newWidth = getWidthFromDelta({ deltaX, maxWidth })
        if (newWidth !== width) {
          onSizeChangeRef.current({ width: newWidth }, { replace: false })
        } else {
          onSizeChangeRef.current({}, { replace: false })
        }

        setIsWidthDragging(false)
        setPreviewSize(null)
        ownerDocument.body.style.cursor = previousCursor
        ownerDocument.body.style.userSelect = previousUserSelect
        ownerDocument.removeEventListener('mousemove', handleMouseMove)
        ownerDocument.removeEventListener('mouseup', handleMouseUp)
      }

      ownerDocument.addEventListener('mousemove', handleMouseMove)
      ownerDocument.addEventListener('mouseup', handleMouseUp)
    },
    [getMaxWidth, getWidthFromDelta, height, width],
  )

  const handleWidthDoubleClick = React.useCallback(
    (e: React.MouseEvent<HTMLDivElement>) => {
      e.preventDefault()
      e.stopPropagation()
      if (width !== undefined) {
        onSizeChangeRef.current({ width: undefined }, { replace: false })
      }
    },
    [width],
  )

  const handleCornerMouseDown = React.useCallback(
    (e: React.MouseEvent<HTMLDivElement>) => {
      const maxWidth = getMaxWidth()
      if (!maxWidth) return

      e.preventDefault()
      e.stopPropagation()
      setIsHeightDragging(true)
      setIsWidthDragging(true)
      setIsCornerDragging(true)
      startXRef.current = e.clientX
      startYRef.current = e.clientY
      startHeightRef.current = height
      startWidthRef.current = Math.min(width ?? maxWidth, maxWidth)
      setPreviewSize({ height, width: startWidthRef.current })

      const ownerDocument = e.currentTarget.ownerDocument
      const previousCursor = ownerDocument.body.style.cursor
      const previousUserSelect = ownerDocument.body.style.userSelect
      ownerDocument.body.style.cursor = 'nwse-resize'
      ownerDocument.body.style.userSelect = 'none'

      const handleMouseMove = (e: MouseEvent) => {
        const deltaX = e.clientX - startXRef.current
        const deltaY = e.clientY - startYRef.current
        const newHeight = getHeightFromDelta(deltaY)
        const newWidth = getWidthFromDelta({ deltaX, maxWidth })

        setPreviewSize({ height: newHeight, width: newWidth ?? maxWidth })
        onSizeChangeRef.current(
          { height: newHeight, width: newWidth },
          { replace: true },
        )
      }

      const handleMouseUp = (e: MouseEvent) => {
        const deltaX = e.clientX - startXRef.current
        const deltaY = e.clientY - startYRef.current
        const newHeight = getHeightFromDelta(deltaY)
        const newWidth = getWidthFromDelta({ deltaX, maxWidth })
        const hasHeightChanged = newHeight !== startHeightRef.current
        const hasWidthChanged = newWidth !== width

        if (hasHeightChanged || hasWidthChanged) {
          onSizeChangeRef.current(
            {
              ...(hasHeightChanged ? { height: newHeight } : {}),
              ...(hasWidthChanged ? { width: newWidth } : {}),
            },
            { replace: false },
          )
        } else {
          onSizeChangeRef.current({}, { replace: false })
        }

        setIsHeightDragging(false)
        setIsWidthDragging(false)
        setIsCornerDragging(false)
        setPreviewSize(null)
        ownerDocument.body.style.cursor = previousCursor
        ownerDocument.body.style.userSelect = previousUserSelect
        ownerDocument.removeEventListener('mousemove', handleMouseMove)
        ownerDocument.removeEventListener('mouseup', handleMouseUp)
      }

      ownerDocument.addEventListener('mousemove', handleMouseMove)
      ownerDocument.addEventListener('mouseup', handleMouseUp)
    },
    [getHeightFromDelta, getMaxWidth, getWidthFromDelta, height, width],
  )

  const visibleWidth = Math.round(previewSize?.width ?? width ?? containerWidth)
  const visibleHeight = Math.round(previewSize?.height ?? height)
  const widthLabel = `${visibleWidth}px`
  const heightLabel = `${visibleHeight}px`
  const cornerLabel = `${visibleWidth}px x ${visibleHeight}px`
  const showWidthTooltip = isWidthDragging && !isCornerDragging
  const showHeightTooltip = isHeightDragging && !isCornerDragging

  return (
    <div ref={containerRef} className="relative w-full">
      <div
        className="relative mx-auto"
        style={{ width: width ?? '100%', maxWidth: '100%' }}
      >
        {children}
        {enableWidthResize ? (
          <div
            role="separator"
            aria-label="Resize chart width"
            aria-orientation="vertical"
            onMouseDown={handleWidthMouseDown}
            onDoubleClick={handleWidthDoubleClick}
            className="group absolute right-0 top-0 z-20 flex w-3 cursor-ew-resize select-none items-center justify-center hover:bg-gray-500/20"
            style={{ height: Math.max(0, height - 8) }}
          >
            <span
              aria-hidden="true"
              className={`${dimensionTooltipStyles} right-full top-1/2 mr-1 -translate-y-1/2 opacity-0 transition-opacity group-hover:opacity-100 ${
                showWidthTooltip ? 'opacity-100' : ''
              }`}
            >
              {widthLabel}
            </span>
            <div className="h-8 w-1 rounded-full bg-gray-400" />
          </div>
        ) : null}
        {enableWidthResize ? (
          <div
            role="separator"
            aria-label="Resize chart width and height"
            aria-orientation="horizontal"
            onMouseDown={handleCornerMouseDown}
            onDoubleClick={handleWidthDoubleClick}
            className="group absolute right-0 z-40 flex h-5 w-5 cursor-nwse-resize select-none items-end justify-end rounded-tl-md p-1 hover:bg-gray-500/20"
            style={{ top: Math.max(0, height - 20) }}
          >
            <span
              aria-hidden="true"
              className={`${dimensionTooltipStyles} bottom-full right-full mb-1 mr-1 opacity-0 transition-opacity group-hover:opacity-100 ${
                isCornerDragging ? 'opacity-100' : ''
              }`}
            >
              {cornerLabel}
            </span>
            <div className="h-2.5 w-2.5 border-b-2 border-r-2 border-gray-400" />
          </div>
        ) : null}
        <div
          role="separator"
          aria-label="Resize chart height"
          aria-orientation="horizontal"
          onMouseDown={handleHeightMouseDown}
          className="group absolute left-0 right-0 z-30 flex h-2 cursor-ns-resize select-none items-center justify-center hover:bg-gray-500/20"
          style={{ top: Math.max(0, height - 2) }}
        >
          <span
            aria-hidden="true"
            className={`${dimensionTooltipStyles} bottom-full left-1/2 mb-1 -translate-x-1/2 opacity-0 transition-opacity group-hover:opacity-100 ${
              showHeightTooltip ? 'opacity-100' : ''
            }`}
          >
            {heightLabel}
          </span>
          <div className="h-1 w-8 rounded-full bg-gray-400" />
        </div>
      </div>
    </div>
  )
}
````

## File: src/components/npm-stats/shared.ts
````typescript
import * as v from 'valibot'

import { packageGroupSchema } from '~/routes/stats/npm/-comparisons'
import { defaultColors } from '~/utils/npm-packages'

export type PackageGroup = v.InferOutput<typeof packageGroupSchema>

export function getPackageGroupLabel(packageGroup: PackageGroup): string {
  return packageGroup.label?.trim() || packageGroup.packages[0]?.name || ''
}

export function hasPackageGroupLabel(packageGroup: PackageGroup): boolean {
  return !!packageGroup.label?.trim()
}

export function isPackageGroupHidden(packageGroup: PackageGroup): boolean {
  if (packageGroup.hidden !== undefined) return packageGroup.hidden

  if (hasPackageGroupLabel(packageGroup) && !packageGroup.baseline) {
    return false
  }

  return !!packageGroup.packages[0]?.hidden
}

export const binTypeSchema = v.picklist([
  'yearly',
  'monthly',
  'weekly',
  'daily',
])
export type BinType = v.InferOutput<typeof binTypeSchema>

export const transformModeSchema = v.picklist(['none', 'normalize-y'])
export type TransformMode = v.InferOutput<typeof transformModeSchema>

export const showDataModeSchema = v.picklist(['all', 'complete'])
export type ShowDataMode = v.InferOutput<typeof showDataModeSchema>

export const latestBarSortSchema = v.picklist(['name', 'value'])
export type LatestBarSort = v.InferOutput<typeof latestBarSortSchema>

export const defaultPlaybackIntervalMs = 350
export const maxPlaybackIntervalMs = 60000
export const playbackIntervalMsSchema = v.pipe(
  v.number(),
  v.integer(),
  v.minValue(1),
  v.maxValue(maxPlaybackIntervalMs),
)

export const barOrientationSchema = v.picklist(['vertical', 'horizontal'])
export type BarOrientation = v.InferOutput<typeof barOrientationSchema>

export const viewModeSchema = v.picklist(['history', 'latest'])
export type ViewMode = v.InferOutput<typeof viewModeSchema>

export const chartTypeSchema = v.picklist([
  'line',
  'stacked',
  'stacked-area',
  'stacked-stream',
  'bar',
  'horizontal-bar',
  'stacked-bar',
])
export type ChartType = v.InferOutput<typeof chartTypeSchema>

export type TimeRange =
  | '7-days'
  | '30-days'
  | '90-days'
  | '180-days'
  | '365-days'
  | '730-days'
  | '1825-days'
  | 'all-time'

export type NpmQueryData = Array<{
  packages: Array<{
    name?: string
    hidden?: boolean
    downloads: Array<{ day: string; downloads: number }>
  }>
  start: string
  end: string
  error?: string
  actualStartDate?: Date
}>

export const binningOptions = [
  {
    label: 'Yearly',
    value: 'yearly',
    single: 'year',
  },
  {
    label: 'Monthly',
    value: 'monthly',
    single: 'month',
  },
  {
    label: 'Weekly',
    value: 'weekly',
    single: 'week',
  },
  {
    label: 'Daily',
    value: 'daily',
    single: 'day',
  },
] as const

export const historyChartTypes = [
  { label: 'Line', value: 'line' },
  { label: 'Stacked', value: 'stacked' },
  { label: 'Stacked Area', value: 'stacked-area' },
  { label: 'Stream', value: 'stacked-stream' },
] as const satisfies ReadonlyArray<{
  label: string
  value: ChartType
}>

export const latestChartTypes = [
  { label: 'Bar', value: 'bar' },
  { label: 'Stacked Bar', value: 'stacked-bar' },
] as const satisfies ReadonlyArray<{
  label: string
  value: ChartType
}>

export const timeRanges = [
  { value: '7-days', label: '7 Days' },
  { value: '30-days', label: '30 Days' },
  { value: '90-days', label: '90 Days' },
  { value: '180-days', label: '6 Months' },
  { value: '365-days', label: '1 Year' },
  { value: '730-days', label: '2 Years' },
  { value: '1825-days', label: '5 Years' },
  { value: 'all-time', label: 'All Time' },
] as const

export const defaultRangeBinTypes: Record<TimeRange, BinType> = {
  '7-days': 'daily',
  '30-days': 'daily',
  '90-days': 'weekly',
  '180-days': 'weekly',
  '365-days': 'weekly',
  '730-days': 'weekly',
  '1825-days': 'weekly',
  'all-time': 'weekly',
}

export function getPackageColor(
  packageName: string,
  packages: PackageGroup[],
): string {
  const packageInfo = packages.find(
    (pkg) =>
      pkg.label === packageName ||
      pkg.packages.some((p) => p.name === packageName),
  )

  if (packageInfo?.color) {
    return packageInfo.color
  }

  const packageIndex = packages.findIndex(
    (pkg) =>
      pkg.label === packageName ||
      pkg.packages.some((p) => p.name === packageName),
  )

  return defaultColors[Math.max(packageIndex, 0) % defaultColors.length]
}

export function getBaselineDisplayName(packageGroups: PackageGroup[]): string {
  const baselineNames = packageGroups
    .filter((packageGroup) => packageGroup.baseline)
    .flatMap((packageGroup) => {
      const label = packageGroup.baselineLabel?.trim()
      if (label) return [label]

      const groupLabel = getPackageGroupLabel(packageGroup)
      return groupLabel ? [groupLabel] : []
    })

  return [...new Set(baselineNames)].join(', ') || 'Baseline'
}

export const formatNumber = (num: number) => {
  if (num >= 1_000_000) {
    return `${(num / 1_000_000).toFixed(1)}M`
  }
  if (num >= 1_000) {
    return `${(num / 1_000).toFixed(1)}k`
  }
  return num.toString()
}

export function isBinningOptionValidForRange(
  range: TimeRange,
  binType: BinType,
): boolean {
  switch (range) {
    case '7-days':
    case '30-days':
      return binType === 'daily'
    case '90-days':
    case '180-days':
    case '365-days':
      return (
        binType === 'daily' || binType === 'weekly' || binType === 'monthly'
      )
    case '730-days':
    case '1825-days':
    case 'all-time':
      return true
  }
}

export function isChartTypeValidForViewMode(
  viewMode: ViewMode,
  chartType: ChartType,
): boolean {
  const options = viewMode === 'history' ? historyChartTypes : latestChartTypes
  return options.some((option) => option.value === chartType)
}

export function getDefaultChartTypeForViewMode(viewMode: ViewMode): ChartType {
  return viewMode === 'history' ? 'line' : 'bar'
}
````

## File: src/components/npm-stats/StatsTable.tsx
````typescript
import * as React from 'react'
import * as d3 from 'd3'
import { XIcon, EyeSlashIcon } from '@phosphor-icons/react'
import { Tooltip } from '~/components/Tooltip'
import { binningOptionsByType } from './binning'
import {
  type PackageGroup,
  type TransformMode,
  type NpmQueryData,
  type BinType,
  type ViewMode,
  getPackageColor,
  getPackageGroupLabel,
  hasPackageGroupLabel,
  isPackageGroupHidden,
  formatNumber,
} from './shared'

export interface StatsTableProps {
  queryData: NpmQueryData | undefined
  packageGroups: PackageGroup[]
  binType: BinType
  transform: TransformMode
  viewMode?: ViewMode
  packageGroupIndexes?: number[]
  onColorClick: (packageName: string, event: React.MouseEvent) => void
  onToggleVisibility: (index: number, packageName: string) => void
  onRemove: (index: number) => void
}

interface PackageStat {
  package: string
  packageName: string
  packages: Array<{
    hidden: boolean | undefined
    name: string
  }>
  totalDownloads: number
  binDownloads: number
  growth: number
  growthPercentage: number
  color: string
  hidden: boolean | undefined
  index: number
}

/**
 * Calculates package statistics from query data for display in the stats table.
 */
function calculateStats(
  queryData: NpmQueryData | undefined,
  packageGroups: PackageGroup[],
  binType: BinType,
  viewMode: ViewMode,
  packageGroupIndexes: number[] | undefined,
): PackageStat[] {
  if (!queryData) return []

  const binOption = binningOptionsByType[binType]

  return queryData
    .map((packageGroupDownloads, index) => {
      if (packageGroups[index]?.baseline) return null
      if (!packageGroupDownloads.packages.some((p) => p.downloads.length)) {
        return null
      }

      const packageGroup = packageGroups[index]
      const firstPackage = packageGroupDownloads.packages[0]
      if (!firstPackage?.name) return null

      const shouldAlwaysIncludeFirstPackage =
        !packageGroup ||
        !hasPackageGroupLabel(packageGroup) ||
        !!packageGroup.baseline

      const visiblePackages = packageGroupDownloads.packages.filter((p, i) => {
        const hiddenState = packageGroup?.packages.find(
          (pg) => pg.name === p.name,
        )?.hidden
        return (i === 0 && shouldAlwaysIncludeFirstPackage) || !hiddenState
      })

      const sortedDownloads = visiblePackages
        .flatMap((p) => p.downloads)
        .sort(
          (a, b) =>
            d3.utcDay(new Date(a.day)).getTime() -
            d3.utcDay(new Date(b.day)).getTime(),
        )

      const binUnit = binOption.bin
      const now = d3.utcDay(new Date())
      const partialBinEnd = binUnit.floor(now)

      const filteredDownloads =
        viewMode === 'latest'
          ? sortedDownloads
          : sortedDownloads.filter(
              (d) => d3.utcDay(new Date(d.day)) < partialBinEnd,
            )

      const binnedDownloads = d3.sort(
        d3.rollup(
          filteredDownloads,
          (v) => d3.sum(v, (d) => d.downloads),
          (d) => binUnit.floor(new Date(d.day)),
        ),
        (d) => d[0],
      )

      const color = getPackageColor(firstPackage.name, packageGroups)

      const firstBin = binnedDownloads[0]
      const lastBin = binnedDownloads[binnedDownloads.length - 1]

      if (!firstBin || !lastBin) return null

      const growth = lastBin[1] - firstBin[1]
      const growthPercentage = growth / (firstBin[1] || 1)

      return {
        package: packageGroup
          ? getPackageGroupLabel(packageGroup)
          : firstPackage.name,
        packageName:
          packageGroup && hasPackageGroupLabel(packageGroup)
            ? getPackageGroupLabel(packageGroup)
            : firstPackage.name,
        packages: packageGroup?.packages.map((pkg) => ({
          hidden: pkg.hidden,
          name: pkg.name,
        })) ?? [{ hidden: firstPackage.hidden, name: firstPackage.name }],
        totalDownloads: d3.sum(binnedDownloads, (d) => d[1]),
        binDownloads: lastBin[1],
        growth,
        growthPercentage,
        color,
        hidden: packageGroup ? isPackageGroupHidden(packageGroup) : undefined,
        index: packageGroupIndexes?.[index] ?? index,
      }
    })
    .filter((stat): stat is PackageStat => stat != null)
}

/**
 * A table showing download statistics for packages.
 * Displays package name, total downloads, and downloads per period.
 */
export function StatsTable({
  queryData,
  packageGroups,
  binType,
  transform,
  viewMode = 'history',
  packageGroupIndexes,
  onColorClick,
  onToggleVisibility,
  onRemove,
}: StatsTableProps) {
  const binOption = binningOptionsByType[binType]
  const stats = calculateStats(
    queryData,
    packageGroups,
    binType,
    viewMode,
    packageGroupIndexes,
  )
  const sortedStats = [...stats].sort((a, b) =>
    transform === 'normalize-y'
      ? b.growth - a.growth
      : b.binDownloads - a.binDownloads,
  )
  const showPackagesColumn = packageGroups.some(
    (packageGroup) =>
      !packageGroup.baseline &&
      (hasPackageGroupLabel(packageGroup) || packageGroup.packages.length > 1),
  )

  return (
    <div className="overflow-x-auto rounded-xl">
      <table className="min-w-full">
        <thead className="bg-gray-500/10">
          <tr>
            <th className="px-3 sm:px-6 py-2 sm:py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">
              Package Name
            </th>
            {showPackagesColumn && (
              <th className="px-3 sm:px-6 py-2 sm:py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">
                NPM Packages
              </th>
            )}
            <th className="px-3 sm:px-6 py-2 sm:py-3 text-right text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">
              {viewMode === 'latest'
                ? 'Selected Bucket Downloads'
                : 'Total Period Downloads'}
            </th>
            <th className="px-3 sm:px-6 py-2 sm:py-3 text-right text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">
              Downloads {viewMode === 'latest' ? 'selected' : 'last'}{' '}
              {binOption.single}
            </th>
          </tr>
        </thead>
        <tbody className="bg-gray-500/5 divide-y divide-gray-500/10">
          {sortedStats.map((stat) => (
            <tr key={stat.package}>
              <td className="px-3 sm:px-6 py-1 sm:py-2 whitespace-nowrap text-xs sm:text-sm font-medium text-gray-900 dark:text-gray-100">
                <div className="flex items-center gap-2">
                  <Tooltip content="Change color">
                    <button
                      onClick={(e) => onColorClick(stat.packageName, e)}
                      className="hover:opacity-80"
                    >
                      <div
                        className="w-4 h-4 rounded"
                        style={{ backgroundColor: stat.color }}
                      />
                    </button>
                  </Tooltip>
                  <div className="flex items-center gap-1">
                    <div className="flex items-center gap-0.5">
                      <Tooltip content="Toggle visibility">
                        <button
                          onClick={() =>
                            onToggleVisibility(stat.index, stat.packageName)
                          }
                          className="p-0.5 hover:text-blue-500 flex items-center gap-1"
                        >
                          <span className={stat.hidden ? 'opacity-50' : ''}>
                            {stat.package}
                          </span>
                          {stat.hidden ? <EyeSlashIcon className="" /> : null}
                        </button>
                      </Tooltip>
                      <Tooltip content="Remove package">
                        <button
                          onClick={() => onRemove(stat.index)}
                          className="p-0.5 text-gray-500 hover:text-red-500"
                        >
                          <XIcon className="" />
                        </button>
                      </Tooltip>
                    </div>
                  </div>
                </div>
              </td>
              {showPackagesColumn && (
                <td className="px-3 sm:px-6 py-1 sm:py-2 text-xs sm:text-sm text-gray-500 dark:text-gray-400">
                  <div className="flex flex-wrap gap-1">
                    {stat.packages.slice(0, 4).map((pkg) => (
                      <span
                        key={pkg.name}
                        className={`rounded bg-gray-500/10 px-1.5 py-0.5 leading-none ${pkg.hidden ? 'opacity-50' : ''}`}
                      >
                        {pkg.name}
                      </span>
                    ))}
                    {stat.packages.length > 4 && (
                      <span className="rounded bg-gray-500/10 px-1.5 py-0.5 leading-none">
                        + {stat.packages.length - 4} more
                      </span>
                    )}
                  </div>
                </td>
              )}
              <td className="px-3 sm:px-6 py-1 sm:py-2 whitespace-nowrap text-xs sm:text-sm text-gray-500 dark:text-gray-400 text-right">
                {formatNumber(stat.totalDownloads)}
              </td>
              <td className="px-3 sm:px-6 py-1 sm:py-2 whitespace-nowrap text-xs sm:text-sm text-gray-500 dark:text-gray-400 text-right">
                {formatNumber(stat.binDownloads)}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
````

## File: src/components/npm-stats/svgDimensions.ts
````typescript
// SVGLength.SVG_LENGTHTYPE_PERCENTAGE. Inlined so this module does not depend on
// the SVGLength global (unavailable outside the browser).
const SVG_LENGTHTYPE_PERCENTAGE = 2

type AnimatedLengthLike = {
  baseVal: { unitType: number; value: number }
}

type SvgLike = {
  width: AnimatedLengthLike
  height: AnimatedLengthLike
  viewBox: { baseVal: { width: number; height: number } | null }
  getBoundingClientRect: () => { width: number; height: number }
}

function isNotSupportedError(error: unknown) {
  return (
    typeof error === 'object' &&
    error !== null &&
    'name' in error &&
    error.name === 'NotSupportedError'
  )
}

/**
 * Reads an absolute SVG length in user units, returning 0 when the length is
 * relative (e.g. `width="100%"`). Browsers throw `NotSupportedError` when a
 * relative length cannot be resolved, which happens for SVGs that are not
 * attached to the document (such as off-screen animated export frames).
 */
export function readAbsoluteSvgLength(length: AnimatedLengthLike) {
  const baseVal = length.baseVal
  if (baseVal.unitType === SVG_LENGTHTYPE_PERCENTAGE) return 0

  try {
    return baseVal.value
  } catch (error) {
    if (isNotSupportedError(error)) return 0
    throw error
  }
}

export function getSvgDimensions(svg: SvgLike) {
  const box = svg.getBoundingClientRect()
  const viewBox = svg.viewBox.baseVal
  const width =
    readAbsoluteSvgLength(svg.width) || viewBox?.width || box.width || 1
  const height =
    readAbsoluteSvgLength(svg.height) || viewBox?.height || box.height || 1

  return {
    width: Math.max(1, Math.ceil(width)),
    height: Math.max(1, Math.ceil(height)),
  }
}
````

## File: src/routes/stats/npm/-comparisons.ts
````typescript
import * as v from 'valibot'
import { npmPackageNameSchema } from '~/utils/schemas'
import {
  MAX_NPM_STATS_GROUPS,
  MAX_NPM_STATS_PACKAGES_PER_GROUP,
  MAX_NPM_STATS_TOTAL_PACKAGES,
} from '~/utils/npm-stats-limits'
import { tanStackTotalNpmStatsPackageGroup } from '~/utils/tanstack-npm-stats'

export const packageGroupSchema = v.object({
  label: v.optional(v.pipe(v.string(), v.maxLength(80))),
  hidden: v.optional(v.boolean()),
  packages: v.pipe(
    v.array(
      v.object({
        name: npmPackageNameSchema,
        hidden: v.optional(v.boolean()),
      }),
    ),
    v.maxLength(MAX_NPM_STATS_PACKAGES_PER_GROUP),
  ),
  color: v.optional(v.nullable(v.pipe(v.string(), v.maxLength(32)))),
  baseline: v.optional(v.boolean()),
  baselineLabel: v.optional(v.pipe(v.string(), v.maxLength(80))),
})

export const packageGroupsSchema = v.pipe(
  v.array(packageGroupSchema),
  v.maxLength(MAX_NPM_STATS_GROUPS),
  v.check(
    (groups) =>
      groups.reduce((count, group) => count + group.packages.length, 0) <=
      MAX_NPM_STATS_TOTAL_PACKAGES,
    `NPM stats comparisons support up to ${MAX_NPM_STATS_TOTAL_PACKAGES} total packages.`,
  ),
)

export const packageComparisonSchema = v.object({
  title: v.pipe(v.string(), v.maxLength(120)),
  packageGroups: packageGroupsSchema,
  baseline: v.optional(v.string()),
})

type PackageGroupInput = v.InferInput<typeof packageGroupSchema>

export type BaselinePreset = {
  id: string
  title: string
  description: string
  category: 'single' | 'index'
  packages: Array<{ name: string; color?: string }>
}

export function getBaselinePresets(): BaselinePreset[] {
  return [
    // Single-package presets — direct comparison against a familiar reference.
    {
      id: 'react',
      title: 'React',
      description: 'Compare against React downloads.',
      category: 'single',
      packages: [{ name: 'react', color: '#61DAFB' }],
    },
    {
      id: 'vue',
      title: 'Vue',
      description: 'Compare against Vue downloads.',
      category: 'single',
      packages: [{ name: 'vue', color: '#41B883' }],
    },
    {
      id: 'angular',
      title: 'Angular',
      description: 'Compare against Angular core downloads.',
      category: 'single',
      packages: [{ name: '@angular/core', color: '#DD0031' }],
    },
    {
      id: 'svelte',
      title: 'Svelte',
      description: 'Compare against Svelte downloads.',
      category: 'single',
      packages: [{ name: 'svelte', color: '#FF3E00' }],
    },
    {
      id: 'typescript',
      title: 'TypeScript',
      description: 'Compare against TypeScript downloads.',
      category: 'single',
      packages: [{ name: 'typescript', color: '#3178C6' }],
    },
    {
      id: 'lodash',
      title: 'Lodash',
      description: 'Compare against Lodash downloads.',
      category: 'single',
      packages: [{ name: 'lodash', color: '#3492FF' }],
    },
    {
      id: 'express',
      title: 'Express',
      description: 'Compare against Express downloads.',
      category: 'single',
      packages: [{ name: 'express', color: '#000000' }],
    },
    // Multi-package indexes — equal-weighted (each member contributes its
    // growth rate equally regardless of size) so the largest member can't
    // dominate the line shape.
    {
      id: 'npm-ecosystem',
      title: 'NPM Ecosystem (index)',
      description:
        'Equal-weighted growth index across foundational packages: lodash, typescript, express, axios. Each contributes its growth rate equally.',
      category: 'index',
      packages: [
        { name: 'lodash', color: '#3492FF' },
        { name: 'typescript', color: '#3178C6' },
        { name: 'express', color: '#000000' },
        { name: 'axios', color: '#5A29E4' },
      ],
    },
    {
      id: 'frontend-frameworks',
      title: 'Frontend Frameworks (index)',
      description:
        'Equal-weighted growth index across the major UI frameworks: react, vue, angular, svelte, solid-js.',
      category: 'index',
      packages: [
        { name: 'react', color: '#61DAFB' },
        { name: 'vue', color: '#41B883' },
        { name: '@angular/core', color: '#DD0031' },
        { name: 'svelte', color: '#FF3E00' },
        { name: 'solid-js', color: '#2C4F7C' },
      ],
    },
  ]
}

const dataFetchingPackageGroups = [
  {
    packages: [{ name: '@tanstack/react-query' }, { name: 'react-query' }],
    color: '#FF4500',
  },
  {
    packages: [{ name: 'swr' }],
    color: '#ec4899',
  },
  {
    packages: [{ name: '@apollo/client' }],
    color: '#6B46C1',
  },
  {
    packages: [{ name: '@trpc/client' }],
    color: '#2596BE',
  },
] satisfies PackageGroupInput[]

const tanstackLibraryPackageGroups = [
  {
    label: 'TanStack Query',
    packages: [{ name: '@tanstack/query-core' }, { name: 'react-query' }],
    color: '#FF4500',
  },
  {
    label: 'TanStack Table',
    packages: [{ name: '@tanstack/table-core' }, { name: 'react-table' }],
    color: '#FF7043',
  },
  {
    label: 'TanStack Router',
    packages: [{ name: '@tanstack/router-core' }, { name: 'react-location' }],
    color: '#32CD32',
  },
  {
    label: 'TanStack Start',
    packages: [{ name: '@tanstack/start-client-core' }],
    color: '#00CED1',
  },
  {
    label: 'TanStack Form',
    packages: [{ name: '@tanstack/form-core' }],
    color: '#FFD700',
  },
  {
    label: 'TanStack Virtual',
    packages: [{ name: '@tanstack/virtual-core' }, { name: 'react-virtual' }],
    color: '#8B5CF6',
  },
  {
    label: 'TanStack DB',
    packages: [{ name: '@tanstack/db' }],
    color: '#F97316',
  },
  {
    label: 'TanStack Pacer',
    packages: [{ name: '@tanstack/pacer' }, { name: '@tanstack/pacer-lite' }],
    color: '#84CC16',
  },
  {
    label: 'TanStack AI',
    packages: [{ name: '@tanstack/ai' }],
    color: '#EC4899',
  },
  {
    label: 'TanStack Intent',
    packages: [{ name: '@tanstack/intent' }],
    color: '#0EA5E9',
  },
  {
    label: 'TanStack Store',
    packages: [{ name: '@tanstack/store' }],
    color: '#B89A56',
  },
  {
    label: 'TanStack Hotkeys',
    packages: [{ name: '@tanstack/hotkeys' }],
    color: '#F43F5E',
  },
  {
    label: 'TanStack Ranger',
    packages: [{ name: '@tanstack/ranger' }, { name: 'react-ranger' }],
    color: '#1F2937',
  },
  {
    label: 'TanStack Config',
    packages: [{ name: '@tanstack/config' }],
    color: '#1F2937',
  },
  {
    label: 'TanStack Devtools',
    packages: [{ name: '@tanstack/devtools' }],
    color: '#1F2937',
  },
  {
    label: 'TanStack CLI',
    packages: [{ name: '@tanstack/cli' }],
    color: '#6366F1',
  },
] satisfies PackageGroupInput[]

const tanstackAggregatePackageGroup =
  tanStackTotalNpmStatsPackageGroup satisfies PackageGroupInput

const ecosystemTanStackPackageGroups = [
  {
    label: 'TanStack Query',
    packages: [{ name: '@tanstack/query-core' }, { name: 'react-query' }],
    color: '#FF4500',
  },
  {
    label: 'TanStack Table',
    packages: [{ name: '@tanstack/table-core' }, { name: 'react-table' }],
    color: '#FF7043',
  },
  {
    label: 'TanStack Router',
    packages: [{ name: '@tanstack/router-core' }],
    color: '#32CD32',
  },
  {
    label: 'TanStack Start',
    packages: [{ name: '@tanstack/start-client-core' }],
    color: '#00CED1',
  },
  {
    label: 'TanStack Form',
    packages: [{ name: '@tanstack/form-core' }],
    color: '#FFD700',
  },
  {
    label: 'TanStack Virtual',
    packages: [{ name: '@tanstack/virtual-core' }, { name: 'react-virtual' }],
    color: '#8B5CF6',
  },
  {
    label: 'TanStack DB',
    packages: [{ name: '@tanstack/db' }],
    color: '#F97316',
  },
  {
    label: 'TanStack Pacer',
    packages: [{ name: '@tanstack/pacer' }],
    color: '#84CC16',
  },
  {
    label: 'TanStack AI',
    packages: [{ name: '@tanstack/ai' }],
    color: '#EC4899',
  },
  {
    label: 'TanStack Intent',
    packages: [{ name: '@tanstack/intent' }],
    color: '#0EA5E9',
  },
  {
    label: 'TanStack Store',
    packages: [{ name: '@tanstack/store' }],
    color: '#B89A56',
  },
] satisfies PackageGroupInput[]

const ecosystemComparisonPackageGroups = [
  {
    label: 'TanStack',
    packages: [
      { name: '@tanstack/query-core' },
      { name: 'react-query' },
      { name: '@tanstack/table-core' },
      { name: 'react-table' },
      { name: '@tanstack/router-core' },
      { name: '@tanstack/start-client-core' },
      { name: '@tanstack/form-core' },
      { name: '@tanstack/virtual-core' },
      { name: 'react-virtual' },
      { name: '@tanstack/db' },
      { name: '@tanstack/pacer' },
      { name: '@tanstack/ai' },
      { name: '@tanstack/intent' },
      { name: '@tanstack/store', hidden: true },
    ],
    color: '#01a7b9',
  },
  {
    label: 'Next.js',
    packages: [{ name: 'next' }, { name: 'ai' }, { name: 'workflow' }],
    color: '#6d6a6a',
  },
  {
    label: 'React Router',
    packages: [
      { name: 'react-router' },
      { name: '@remix-run/react' },
      { name: 'remix' },
    ],
    color: '#ff7580',
  },
  {
    label: 'Astro',
    packages: [{ name: 'astro' }],
    color: '#BC52EE',
  },
  {
    label: 'Vue',
    packages: [{ name: 'vue' }],
    color: '#6aaf04',
  },
  {
    label: 'Angular',
    packages: [{ name: '@angular/core' }, { name: 'angular' }],
    color: '#DD0031',
  },
  {
    label: 'Expo',
    packages: [{ name: 'expo' }],
    color: '#F59E0B',
  },
  {
    packages: [{ name: 'svelte' }],
  },
  {
    packages: [{ name: 'vite' }],
  },
  {
    label: 'React',
    packages: [{ name: 'react', hidden: true }],
    color: '#61DAFB',
    baseline: true,
    baselineLabel: 'React',
  },
] satisfies PackageGroupInput[]

// Default comparison for route validation.
export const defaultPackageGroups = ecosystemComparisonPackageGroups

const ecosystemFlattenedPackageGroups = [
  ...ecosystemTanStackPackageGroups,
  {
    label: 'Next.js',
    packages: [{ name: 'next' }],
    color: '#6d6a6a',
  },
  {
    label: 'AI SDK',
    packages: [{ name: 'ai' }],
    color: '#111827',
  },
  {
    label: 'Workflow',
    packages: [{ name: 'workflow' }],
    color: '#9CA3AF',
  },
  {
    label: 'React Router',
    packages: [{ name: 'react-router' }],
    color: '#ff7580',
  },
  {
    label: 'Remix React',
    packages: [{ name: '@remix-run/react' }],
    color: '#EC4899',
  },
  {
    label: 'Remix',
    packages: [{ name: 'remix' }],
    color: '#F97316',
  },
  {
    label: 'Astro',
    packages: [{ name: 'astro' }],
    color: '#BC52EE',
  },
  {
    label: 'Vue',
    packages: [{ name: 'vue' }],
    color: '#6aaf04',
  },
  {
    label: 'Angular',
    packages: [{ name: '@angular/core' }, { name: 'angular' }],
    color: '#DD0031',
  },
  {
    label: 'Expo',
    packages: [{ name: 'expo' }],
    color: '#F59E0B',
  },
  {
    label: 'Svelte',
    packages: [{ name: 'svelte' }],
  },
  {
    label: 'Vite',
    packages: [{ name: 'vite' }],
  },
  {
    label: 'React',
    packages: [{ name: 'react', hidden: true }],
    color: '#61DAFB',
    baseline: true,
    baselineLabel: 'React',
  },
] satisfies PackageGroupInput[]

export function getPopularComparisons(): v.InferInput<
  typeof packageComparisonSchema
>[] {
  return [
    {
      title: 'JavaScript Ecosystem',
      packageGroups: ecosystemComparisonPackageGroups,
    },
    {
      title: 'JavaScript Ecosystem (flat)',
      packageGroups: ecosystemFlattenedPackageGroups,
    },
    {
      title: 'TanStack Total',
      packageGroups: [tanstackAggregatePackageGroup],
    },
    {
      title: 'TanStack Libraries',
      packageGroups: tanstackLibraryPackageGroups,
    },
    {
      title: 'Data Fetching',
      packageGroups: dataFetchingPackageGroups,
    },
    {
      title: 'State Management',
      packageGroups: [
        {
          packages: [{ name: 'redux' }],
          color: '#764ABC',
        },
        {
          packages: [{ name: 'mobx' }],
          color: '#FF9955',
        },
        {
          packages: [{ name: 'zustand' }],
          color: '#764ABC',
        },
        {
          packages: [{ name: 'jotai' }],
          color: '#6366f1',
        },
        {
          packages: [{ name: 'valtio' }],
          color: '#FF6B6B',
        },
        {
          packages: [{ name: '@tanstack/store' }],
          color: '#FF69B4',
        },
        {
          packages: [
            { name: '@tanstack/react-query' },
            { name: 'react-query' },
          ],
          color: '#FF4500',
        },
      ],
    },
    {
      title: 'Routing (React)',
      packageGroups: [
        {
          packages: [{ name: 'react-router' }],
          color: '#FF0000',
        },
        {
          packages: [{ name: '@tanstack/react-router' }],
          color: '#32CD32',
        },
        {
          packages: [{ name: 'next' }],
          color: '#4682B4',
        },
        {
          packages: [{ name: 'wouter' }],
          color: '#8b5cf6',
        },
        {
          packages: [{ name: 'expo' }],
          color: '#f59e0b',
        },
      ],
    },
    {
      title: 'AI & Agent Harnesses',
      packageGroups: [
        {
          label: 'TanStack AI',
          packages: [{ name: '@tanstack/ai' }],
          color: '#EC4899',
        },
        {
          label: 'AI SDK',
          packages: [{ name: 'ai' }],
          color: '#111827',
        },
        {
          label: 'LangChain',
          packages: [{ name: 'langchain' }],
          color: '#1C3C3C',
        },
        {
          label: 'LangGraph',
          packages: [{ name: '@langchain/langgraph' }],
          color: '#2E8B57',
        },
        {
          label: 'Mastra',
          packages: [{ name: '@mastra/core' }],
          color: '#7C3AED',
        },
        {
          label: 'OpenAI Agents',
          packages: [{ name: '@openai/agents' }],
          color: '#10A37F',
        },
        {
          label: 'Genkit',
          packages: [{ name: 'genkit' }],
          color: '#4285F4',
        },
        {
          label: 'CopilotKit',
          packages: [{ name: '@copilotkit/react-core' }],
          color: '#F97316',
        },
        {
          label: 'Promptfoo',
          packages: [{ name: 'promptfoo' }],
          color: '#FACC15',
        },
      ],
    },
    {
      title: 'Data Grids',
      packageGroups: [
        {
          packages: [
            { name: 'ag-grid-community' },
            { name: 'ag-grid-enterprise' },
          ],
          color: '#29B6F6',
        },
        {
          packages: [
            { name: '@tanstack/react-table' },
            { name: 'react-table' },
          ],
          color: '#FF7043',
        },
        {
          packages: [{ name: 'handsontable' }],
          color: '#FFCA28',
        },
        {
          packages: [{ name: '@mui/x-data-grid' }, { name: 'mui-datatables' }],
          color: '#1976D2',
        },
        {
          packages: [{ name: 'react-data-grid' }],
          color: '#4CAF50',
        },
      ],
    },
    {
      title: 'Virtualization',
      packageGroups: [
        {
          packages: [{ name: 'react-virtualized' }],
          color: '#FF6B6B',
        },
        {
          packages: [{ name: 'react-window' }],
          color: '#4ECDC4',
        },
        {
          packages: [
            { name: '@tanstack/react-virtual' },
            { name: 'react-virtual' },
          ],
          color: '#FF4500',
        },
        {
          packages: [{ name: 'react-lazyload' }],
          color: '#FFD93D',
        },
        {
          packages: [{ name: 'virtua' }],
          color: '#6C5CE7',
        },
        {
          packages: [{ name: 'react-virtuoso' }],
          color: '#00B894',
        },
      ],
    },
    {
      title: 'Frameworks',
      packageGroups: [
        {
          packages: [{ name: 'react' }],
          color: '#61DAFB',
        },
        {
          packages: [{ name: 'vue' }],
          color: '#41B883',
        },
        {
          packages: [{ name: '@angular/core' }],
          color: '#DD0031',
        },
        {
          packages: [{ name: 'svelte' }],
          color: '#FF3E00',
        },
        {
          packages: [{ name: 'solid-js' }],
          color: '#2C4F7C',
        },
        {
          packages: [{ name: 'preact' }],
          color: '#673AB8',
        },
      ],
    },
    {
      title: 'Styling',
      packageGroups: [
        {
          packages: [{ name: 'tailwindcss' }],
          color: '#06B6D4',
        },
        {
          packages: [{ name: 'bootstrap' }],
          color: '#7952B3',
        },
        {
          packages: [{ name: '@emotion/react' }],
          color: '#D36AC2',
        },
        {
          packages: [{ name: '@stitches/react' }],
          color: '#8b5cf6',
        },
        {
          packages: [{ name: '@vanilla-extract/css' }],
          color: '#FFB6C1',
        },
      ],
    },
    {
      title: 'Build Tools',
      packageGroups: [
        {
          packages: [{ name: 'webpack' }],
          color: '#8DD6F9',
        },
        {
          packages: [{ name: 'vite' }],
          color: '#008000',
        },
        {
          packages: [{ name: 'rollup' }],
          color: '#e80A3F',
        },
        {
          packages: [{ name: 'rolldown' }],
          color: '#FF5733',
        },
        {
          packages: [{ name: 'esbuild' }],
          color: '#FFCF00',
        },
        {
          packages: [{ name: '@swc/core' }],
          color: '#DEAD0F',
        },
        {
          packages: [{ name: 'parcel' }],
          color: '#2D8CFF',
        },
        {
          packages: [{ name: '@rspack/core' }],
          color: '#8DD6F9',
        },
      ],
    },
    {
      title: 'Testing',
      packageGroups: [
        {
          packages: [{ name: 'jest' }],
          color: '#C21325',
        },
        {
          packages: [{ name: 'vitest' }],
          color: '#646CFF',
        },
        {
          packages: [{ name: '@testing-library/react' }],
          color: '#E33332',
        },
        {
          packages: [{ name: 'cypress' }],
          color: '#4A5568',
        },
        {
          packages: [{ name: 'playwright' }],
          color: '#2EAD33',
        },
        {
          packages: [{ name: '@storybook/react' }],
          color: '#FF4785',
        },
      ],
    },
    {
      title: 'Forms',
      packageGroups: [
        {
          packages: [{ name: 'react-hook-form' }],
          color: '#EC5990',
        },
        {
          packages: [{ name: '@tanstack/form-core' }],
          color: '#FFD700',
        },
        {
          packages: [{ name: '@conform-to/dom' }],
          color: '#FF5733',
        },
        {
          packages: [
            { name: '@formisch/react' },
            { name: '@formisch/svelte' },
            { name: '@formisch/solid' },
            { name: '@formisch/vue' },
            { name: '@formisch/preact' },
            { name: '@formisch/qwik' },
          ],
          color: '#8B5CF6',
        },
      ],
    },
    {
      title: 'UI Components',
      packageGroups: [
        {
          packages: [{ name: '@mui/material' }],
          color: '#0081CB',
        },
        {
          packages: [{ name: '@chakra-ui/react' }],
          color: '#319795',
        },
        {
          packages: [{ name: '@radix-ui/themes' }],
          color: '#FF6F61',
        },
        {
          packages: [{ name: '@headlessui/react' }],
          color: '#f43f5e',
        },
        {
          packages: [{ name: '@mantine/core' }],
          color: '#FFD700',
        },
      ],
    },
    {
      title: 'Animation',
      packageGroups: [
        {
          packages: [{ name: 'motion' }, { name: 'framer-motion' }],
          color: '#FF0055',
        },
        {
          packages: [{ name: 'react-spring' }],
          color: '#FF7F50',
        },
        {
          packages: [{ name: '@react-spring/web' }],
          color: '#FF4500',
        },
        {
          packages: [{ name: 'gsap' }],
          color: '#32CD32',
        },
        {
          packages: [{ name: '@motionone/dom' }],
          color: '#FF1493',
        },
        {
          packages: [{ name: '@formkit/auto-animate' }],
          color: '#FFD700',
        },
      ],
    },
    {
      title: 'Date & Time',
      packageGroups: [
        {
          packages: [{ name: 'date-fns' }],
          color: '#E91E63',
        },
        {
          packages: [{ name: 'dayjs' }],
          color: '#FF6B6B',
        },
        {
          packages: [{ name: 'luxon' }],
          color: '#3498DB',
        },
        {
          packages: [{ name: 'moment' }],
          color: '#4A5568',
        },
        {
          packages: [{ name: '@date-io/date-fns' }],
          color: '#FFD700',
        },
        {
          packages: [{ name: 'temporal-polyfill' }],
          color: '#a855f7',
        },
      ],
    },
    {
      title: 'Validation',
      packageGroups: [
        {
          packages: [{ name: 'zod' }],
          color: '#ef4444',
        },
        {
          packages: [{ name: 'io-ts' }],
          color: '#3b82f6',
        },
        {
          packages: [{ name: 'arktype' }],
          color: '#10b981',
        },
        {
          packages: [{ name: 'valibot' }],
          color: '#f97316',
        },
        {
          packages: [{ name: 'yup' }],
          color: '#06b6d4',
        },
        {
          packages: [{ name: 'joi' }],
          color: '#8b5cf6',
        },
        {
          packages: [{ name: '@sinclair/typebox' }],
          color: '#d946ef',
        },
      ],
    },
    {
      title: 'Documentation',
      packageGroups: [
        {
          packages: [{ name: 'vitepress' }],
          color: '#a8b1ff',
        },
        {
          packages: [{ name: '@docusaurus/core' }],
          color: '#21b091',
        },
        {
          packages: [{ name: 'vocs' }],
          color: '#0090ff',
        },
        {
          packages: [{ name: '@astrojs/starlight' }],
          color: '#f97316',
        },
        {
          packages: [{ name: 'nextra' }],
          color: '#00CED1',
        },
        {
          packages: [{ name: 'fumadocs-core' }],
          color: '#DA70D6',
        },
      ],
    },
    {
      title: 'Drag & Drop',
      packageGroups: [
        {
          label: 'dnd-kit',
          packages: [{ name: '@dnd-kit/core' }],
          color: '#eb2f06',
        },
        {
          label: 'Pragmatic DnD',
          packages: [{ name: '@atlaskit/pragmatic-drag-and-drop' }],
          color: '#0652dd',
        },
        {
          label: 'React DnD',
          packages: [{ name: 'react-dnd' }],
          color: '#10ac84',
        },
        {
          label: 'SortableJS',
          packages: [
            { name: 'sortablejs' },
            { name: 'react-sortablejs' },
            { name: 'vuedraggable' },
            { name: 'vue-draggable-next' },
            { name: 'vue-draggable-plus' },
          ],
          color: '#f39c12',
        },
        {
          label: 'Hello Pangea DnD',
          packages: [
            { name: '@hello-pangea/dnd' },
            { name: 'react-beautiful-dnd' },
          ],
          color: '#8854d0',
        },
        {
          label: 'React Draggable',
          packages: [{ name: 'react-draggable' }],
          color: '#20bf6b',
        },
        {
          label: 'React Grid Layout',
          packages: [{ name: 'react-grid-layout' }],
          color: '#d81b60',
        },
        {
          label: 'React Rnd',
          packages: [{ name: 'react-rnd' }],
          color: '#a55d35',
        },
      ],
    },
  ] as const
}
````

## File: src/routes/stats/npm/-utils.ts
````typescript
import * as v from 'valibot'
import { MAX_NPM_STATS_GROUPS } from '~/utils/npm-stats-limits'

// Re-export shared types and schema without importing chart utilities.
export { binTypeSchema, type BinType } from '~/components/npm-stats/shared'
export {
  chartTypeSchema,
  type ChartType,
  barOrientationSchema,
  type BarOrientation,
  latestBarSortSchema,
  type LatestBarSort,
  defaultPlaybackIntervalMs,
  playbackIntervalMsSchema,
  viewModeSchema,
  type ViewMode,
} from '~/components/npm-stats/shared'

// NPM-specific schemas (extends shared with additional time ranges)
export const transformModeSchema = v.picklist(['none', 'normalize-y'])
export const showDataModeSchema = v.picklist(['all', 'complete'])

export type TransformMode = v.InferOutput<typeof transformModeSchema>
export type ShowDataMode = v.InferOutput<typeof showDataModeSchema>

// NPM stats has additional time ranges (730 days, 1825 days) beyond shared chart utils
export const timeRangeSchema = v.picklist([
  '7-days',
  '30-days',
  '90-days',
  '180-days',
  '365-days',
  '730-days',
  '1825-days',
  'all-time',
])

export type TimeRange = v.InferOutput<typeof timeRangeSchema>

// URL encoding/decoding for package names
// @tanstack/react-query -> @tanstack~1react-query
export function encodePackageName(packageName: string): string {
  return packageName.replace(/~/g, '~0').replace(/\//g, '~1')
}

// @tanstack~1react-query -> @tanstack/react-query
export function decodePackageName(urlName: string): string {
  try {
    const decoded = decodeURIComponent(urlName)
    if (
      decoded.startsWith('@') &&
      decoded.includes('__') &&
      !decoded.includes('/')
    ) {
      return decoded.replace('__', '/')
    }
    return decoded.replace(/~1/g, '/').replace(/~0/g, '~')
  } catch {
    return ''
  }
}

// Parse packages from URL like "react-vs-vue" or "@tanstack__react-query-vs-swr"
export function parsePackagesFromUrl(packagesParam: string): string[] {
  const parts = packagesParam.split('-vs-').slice(0, MAX_NPM_STATS_GROUPS)
  return parts
    .map(decodePackageName)
    .filter((packageName) =>
      /^(?:@[a-z0-9][a-z0-9._-]*\/)?[a-z0-9][a-z0-9._-]*$/.test(packageName),
    )
}

// Generate a comparison URL from package names
export function generateComparisonUrl(packages: string[]): string {
  return packages.map(encodePackageName).join('-vs-')
}

// Generate the full path for a comparison
export function getComparisonPath(packages: string[]): string {
  return `/stats/npm/${generateComparisonUrl(packages)}`
}
````

## File: src/routes/stats/npm/$packages.tsx
````typescript
import {
  createFileRoute,
  notFound,
  redirect,
  stripSearchParams,
} from '@tanstack/react-router'
import * as v from 'valibot'
import {
  parsePackagesFromUrl,
  transformModeSchema,
  binTypeSchema,
  chartTypeSchema,
  barOrientationSchema,
  latestBarSortSchema,
  defaultPlaybackIntervalMs,
  playbackIntervalMsSchema,
  showDataModeSchema,
  timeRangeSchema,
  viewModeSchema,
} from './-utils'
import {
  chartHeightSchema,
  chartWidthSchema,
  isNpmPackageName,
} from '~/utils/schemas'

const timelineZoomTimeSchema = v.pipe(v.number(), v.integer(), v.minValue(0))

export const comparisonSearchSchema = v.object({
  range: v.fallback(v.optional(timeRangeSchema, '365-days'), '365-days'),
  transform: v.fallback(v.optional(transformModeSchema, 'none'), 'none'),
  binType: v.fallback(v.optional(binTypeSchema, 'weekly'), 'weekly'),
  viewMode: v.fallback(v.optional(viewModeSchema, 'history'), 'history'),
  chartType: v.fallback(v.optional(chartTypeSchema, 'line'), 'line'),
  barOrientation: v.fallback(v.optional(barOrientationSchema), undefined),
  barSort: v.fallback(v.optional(latestBarSortSchema, 'value'), 'value'),
  bucketOffset: v.fallback(
    v.optional(
      v.pipe(v.number(), v.integer(), v.minValue(-5000), v.maxValue(0)),
      0,
    ),
    0,
  ),
  playbackIntervalMs: v.fallback(
    v.optional(playbackIntervalMsSchema, defaultPlaybackIntervalMs),
    defaultPlaybackIntervalMs,
  ),
  playbackLoop: v.fallback(v.optional(v.boolean(), false), false),
  playbackPlaying: v.fallback(v.optional(v.boolean(), false), false),
  showDataMode: v.fallback(v.optional(showDataModeSchema, 'all'), 'all'),
  showLegend: v.fallback(v.optional(v.boolean(), false), false),
  height: v.fallback(v.optional(chartHeightSchema, 400), 400),
  width: v.fallback(v.optional(chartWidthSchema), undefined),
  timelineStart: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
  timelineEnd: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
})

export const Route = createFileRoute('/stats/npm/$packages')({
  validateSearch: comparisonSearchSchema,
  search: {
    middlewares: [stripSearchParams(v.parse(comparisonSearchSchema, {}))],
  },
  beforeLoad: redirectNpmComparison,
})

export function redirectNpmComparison({
  params,
  search,
}: {
  params: { packages: string }
  search: v.InferOutput<typeof comparisonSearchSchema>
}) {
  const packages = parsePackagesFromUrl(params.packages)
  if (
    !packages.length ||
    packages.length !== params.packages.split('-vs-').length ||
    !packages.every(isNpmPackageName)
  ) {
    throw notFound()
  }
  throw redirect({
    to: '/stats/npm',
    search: {
      ...search,
      packageGroups: packages.map((name) => ({ packages: [{ name }] })),
    },
    replace: true,
  })
}
````

## File: src/routes/stats/npm/embed.tsx
````typescript
import * as React from 'react'
import { createFileRoute } from '@tanstack/react-router'
import { useQuery } from '@tanstack/react-query'
import * as v from 'valibot'

import { Spinner } from '~/components/Spinner'
import {
  npmQueryOptions,
  selectLatestBucketQueryData,
} from '~/components/npm-stats/npmQueryOptions'
import { getLatestBucketOffsetBounds } from '~/components/npm-stats/binning'
import {
  barOrientationSchema,
  chartTypeSchema,
  defaultPlaybackIntervalMs,
  defaultRangeBinTypes,
  getDefaultChartTypeForViewMode,
  isChartTypeValidForViewMode,
  latestBarSortSchema,
  playbackIntervalMsSchema,
  type BarOrientation,
  type BinType,
  type ChartType,
  type LatestBarSort,
  type PackageGroup,
  type ShowDataMode,
  type TimeRange,
  type TransformMode,
  type ViewMode,
  viewModeSchema,
} from '~/components/npm-stats/shared'
import { chartWidthSchema } from '~/utils/schemas'
import { seo } from '~/utils/seo'
import { defaultPackageGroups, packageGroupsSchema } from './-comparisons'

const LazyNPMStatsChart = React.lazy(() =>
  import('~/components/npm-stats/NPMStatsChart').then((m) => ({
    default: m.NPMStatsChart,
  })),
)

const transformModeSchema = v.picklist(['none', 'normalize-y'])
const binTypeSchema = v.picklist(['yearly', 'monthly', 'weekly', 'daily'])
const showDataModeSchema = v.picklist(['all', 'complete'])
const bucketOffsetSchema = v.pipe(
  v.number(),
  v.integer(),
  v.minValue(-5000),
  v.maxValue(0),
)
const timelineZoomTimeSchema = v.pipe(v.number(), v.integer(), v.minValue(0))

export const Route = createFileRoute('/stats/npm/embed')({
  validateSearch: v.object({
    packageGroups: v.fallback(
      v.optional(packageGroupsSchema, defaultPackageGroups),
      defaultPackageGroups,
    ),
    range: v.fallback(
      v.optional(
        v.picklist([
          '7-days',
          '30-days',
          '90-days',
          '180-days',
          '365-days',
          '730-days',
          '1825-days',
          'all-time',
        ]),
        '1825-days',
      ),
      '1825-days',
    ),
    transform: v.fallback(v.optional(transformModeSchema, 'none'), 'none'),
    binType: v.fallback(v.optional(binTypeSchema, 'weekly'), 'weekly'),
    viewMode: v.fallback(v.optional(viewModeSchema, 'history'), 'history'),
    chartType: v.fallback(v.optional(chartTypeSchema, 'line'), 'line'),
    barOrientation: v.fallback(v.optional(barOrientationSchema), undefined),
    barSort: v.fallback(v.optional(latestBarSortSchema, 'value'), 'value'),
    bucketOffset: v.fallback(v.optional(bucketOffsetSchema, 0), 0),
    playbackIntervalMs: v.fallback(
      v.optional(playbackIntervalMsSchema, defaultPlaybackIntervalMs),
      defaultPlaybackIntervalMs,
    ),
    showDataMode: v.fallback(v.optional(showDataModeSchema, 'all'), 'all'),
    normalizeBaseline: v.fallback(v.optional(v.boolean(), true), true),
    showBaseline: v.fallback(v.optional(v.boolean(), true), true),
    showLegend: v.fallback(v.optional(v.boolean(), false), false),
    width: v.fallback(v.optional(chartWidthSchema), undefined),
    timelineStart: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
    timelineEnd: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
  }),
  head: () => ({
    meta: seo({
      title: 'TanStack NPM Stats Chart Embed',
      description: 'Embeddable npm download statistics chart from TanStack.',
      noindex: true,
    }),
  }),
  component: NpmStatsEmbed,
  staticData: {
    baseParent: true,
    includeSearchInCanonical: true,
    showNavbar: false,
  },
})

type NpmStatsEmbedSearch = {
  packageGroups?: PackageGroup[]
  range?: TimeRange
  transform?: TransformMode
  binType?: BinType
  viewMode?: ViewMode
  chartType?: ChartType
  barOrientation?: BarOrientation
  barSort?: LatestBarSort
  bucketOffset?: number
  playbackIntervalMs?: number
  showDataMode?: ShowDataMode
  normalizeBaseline?: boolean
  showBaseline?: boolean
  showLegend?: boolean
  width?: number
  timelineStart?: number
  timelineEnd?: number
}

function useElementHeight() {
  const containerRef = React.useRef<HTMLDivElement>(null)
  const [height, setHeight] = React.useState(0)

  React.useEffect(() => {
    const container = containerRef.current
    if (!container) return

    const setMeasuredHeight = (nextHeight: number) => {
      setHeight((previousHeight) =>
        previousHeight === nextHeight ? previousHeight : nextHeight,
      )
    }

    const updateHeight = () => {
      setMeasuredHeight(Math.round(container.getBoundingClientRect().height))
    }

    updateHeight()

    const ownerWindow = container.ownerDocument.defaultView
    if (!ownerWindow?.ResizeObserver) {
      ownerWindow?.addEventListener('resize', updateHeight)
      return () => {
        ownerWindow?.removeEventListener('resize', updateHeight)
      }
    }

    const resizeObserver = new ownerWindow.ResizeObserver((entries) => {
      const entry = entries[0]
      if (!entry) return

      setMeasuredHeight(Math.round(entry.contentRect.height))
    })

    resizeObserver.observe(container)

    return () => {
      resizeObserver.disconnect()
    }
  }, [])

  return { containerRef, height }
}

function ChartFallback({ height }: { height: number }) {
  return (
    <div
      className="animate-pulse rounded bg-gray-100 dark:bg-gray-900"
      style={{ height }}
    />
  )
}

function NpmStatsEmbed() {
  const search = Route.useSearch()
  const navigate = Route.useNavigate()
  const { containerRef, height: measuredHeight } = useElementHeight()

  const packageGroups: PackageGroup[] = search.packageGroups ?? []
  const range: TimeRange = search.range ?? '1825-days'
  const transform: TransformMode = search.transform ?? 'none'
  const binTypeParam: BinType | undefined = search.binType
  const viewMode: ViewMode = search.viewMode ?? 'history'
  const chartTypeParam: ChartType = search.chartType ?? 'line'
  const barOrientationParam: BarOrientation =
    search.barOrientation ??
    (chartTypeParam === 'horizontal-bar' ? 'horizontal' : 'vertical')
  const barSort: LatestBarSort = search.barSort ?? 'value'
  const showDataMode: ShowDataMode = search.showDataMode ?? 'all'
  const normalizeBaseline: boolean = search.normalizeBaseline ?? true
  const showBaseline: boolean = search.showBaseline ?? true
  const showLegend: boolean = search.showLegend ?? false
  const bucketOffsetParam: number = search.bucketOffset ?? 0
  const playbackIntervalMs: number =
    search.playbackIntervalMs ?? defaultPlaybackIntervalMs
  const timelineStart: number | undefined = search.timelineStart
  const timelineEnd: number | undefined = search.timelineEnd
  const width: number | undefined = search.width

  const binType: BinType = binTypeParam ?? defaultRangeBinTypes[range]
  const chartType = isChartTypeValidForViewMode(viewMode, chartTypeParam)
    ? chartTypeParam
    : getDefaultChartTypeForViewMode(viewMode)
  const barOrientation: BarOrientation =
    viewMode === 'latest' &&
    (chartType === 'bar' || chartType === 'stacked-bar')
      ? barOrientationParam
      : 'vertical'
  const latestBucketBounds = getLatestBucketOffsetBounds({ binType, range })
  const bucketOffset = Math.max(
    latestBucketBounds.minOffset,
    Math.min(bucketOffsetParam, latestBucketBounds.maxOffset),
  )
  const queryPackageGroups = packageGroups.filter(
    (packageGroup) => viewMode === 'history' || !packageGroup.baseline,
  )

  const npmQuery = useQuery(
    npmQueryOptions({
      packageGroups: queryPackageGroups,
      range,
      viewMode,
      binType,
      bucketOffset,
    }),
  )
  const displayQueryData = React.useMemo(
    () =>
      viewMode === 'latest'
        ? selectLatestBucketQueryData({
            queryData: npmQuery.data,
            binType,
            bucketOffset,
          })
        : npmQuery.data,
    [binType, bucketOffset, npmQuery.data, viewMode],
  )

  const handleTimelineRangeChange = React.useCallback(
    (nextRange: { end: number | undefined; start: number | undefined }) => {
      navigate({
        to: '.',
        search: (prev: NpmStatsEmbedSearch) => ({
          ...prev,
          timelineStart: nextRange.start,
          timelineEnd: nextRange.end,
        }),
        replace: true,
        resetScroll: false,
      })
    },
    [navigate],
  )

  const chartHeight = measuredHeight || 400
  const chartWidthStyle = width
    ? {
        maxWidth: '100%',
        width,
      }
    : undefined

  return (
    <div className="relative flex h-dvh w-screen overflow-hidden bg-white text-gray-950 dark:bg-gray-950 dark:text-gray-50">
      <style
        dangerouslySetInnerHTML={{
          __html: `
html, body {
  background: transparent !important;
  height: 100%;
  margin: 0 !important;
  overflow: hidden;
  width: 100%;
}
`,
        }}
      />
      <div
        className={
          width
            ? 'min-h-0 min-w-0 flex-1'
            : 'min-h-0 min-w-0 flex-1 self-stretch'
        }
        ref={containerRef}
        style={chartWidthStyle}
      >
        {queryPackageGroups.length ? (
          npmQuery.isLoading ? (
            <div
              className="flex items-center justify-center"
              style={{ height: chartHeight }}
            >
              <div className="flex flex-col items-center gap-3">
                <Spinner />
                <div className="text-xs text-gray-600 dark:text-gray-400">
                  Loading download statistics...
                </div>
              </div>
            </div>
          ) : (
            <React.Suspense fallback={<ChartFallback height={chartHeight} />}>
              <LazyNPMStatsChart
                range={range}
                queryData={displayQueryData}
                height={chartHeight}
                viewMode={viewMode}
                chartType={chartType}
                barSort={barSort}
                barOrientation={barOrientation}
                transform={transform}
                binType={binType}
                packages={queryPackageGroups}
                showDataMode={showDataMode}
                normalizeBaseline={viewMode === 'history' && normalizeBaseline}
                showBaseline={viewMode === 'history' && showBaseline}
                showLegend={showLegend}
                timelineStart={timelineStart}
                timelineEnd={timelineEnd}
                animationQueryData={npmQuery.data}
                animationBucketOffsetBounds={latestBucketBounds}
                animationFrameIntervalMs={playbackIntervalMs}
                onTimelineRangeChange={handleTimelineRangeChange}
              />
            </React.Suspense>
          )
        ) : null}
      </div>
      <a
        className="absolute top-1 right-1 z-10 rounded bg-white/75 px-1.5 py-0.5 text-[9px] leading-none font-medium text-gray-500 ring-1 ring-gray-200/80 backdrop-blur hover:text-gray-950 dark:bg-gray-950/75 dark:text-gray-500 dark:ring-gray-800/80 dark:hover:text-gray-50"
        href="/stats/npm"
        rel="noreferrer"
        target="_blank"
      >
        Powered by TanStack NPM Stats
      </a>
    </div>
  )
}
````

## File: src/routes/stats/npm/index.tsx
````typescript
import * as React from 'react'
import { createFileRoute, stripSearchParams } from '@tanstack/react-router'
import * as v from 'valibot'
import { useThrottledCallback, useThrottler } from '@tanstack/react-pacer'
import { QuestionIcon } from '@phosphor-icons/react'
import { useQuery } from '@tanstack/react-query'
import {
  Card,
  Dialog,
  DialogBody,
  DialogContent,
  DialogHeader,
} from '~/components/ds/ui'
import { Tooltip } from '~/components/Tooltip'
import { seo } from '~/utils/seo'
import { chartHeightSchema, chartWidthSchema } from '~/utils/schemas'
import {
  getPopularComparisons,
  getBaselinePresets,
  packageGroupsSchema,
  defaultPackageGroups,
  type BaselinePreset,
} from './-comparisons'
import { Spinner } from '~/components/Spinner'
import { BaselineSection } from '~/components/npm-stats/BaselineSection'
import { ChartControls } from '~/components/npm-stats/ChartControls'
import { ColorPickerPopover } from '~/components/npm-stats/ColorPickerPopover'
import { DisabledChartActions } from '~/components/npm-stats/DisabledChartActions'
import type { NpmStatsChartEmbedOptions } from '~/components/npm-stats/NPMStatsChart'
import { PackagePills } from '~/components/npm-stats/PackagePills'
import { PackageSearch } from '~/components/npm-stats/PackageSearch'
import { PopularComparisons } from '~/components/npm-stats/PopularComparisons'
import {
  Resizable,
  type ResizableSizeChange,
  type ResizeChangeOptions,
} from '~/components/npm-stats/Resizable'
import { StatsTable } from '~/components/npm-stats/StatsTable'
import { LatestBucketNavigator } from '~/components/npm-stats/LatestBucketNavigator'
import {
  npmQueryOptions,
  selectLatestBucketQueryData,
} from '~/components/npm-stats/npmQueryOptions'
import { getLatestBucketOffsetBounds } from '~/components/npm-stats/binning'
import {
  defaultRangeBinTypes,
  getPackageGroupLabel,
  getPackageColor,
  getDefaultChartTypeForViewMode,
  isChartTypeValidForViewMode,
  defaultPlaybackIntervalMs,
  playbackIntervalMsSchema,
  chartTypeSchema,
  barOrientationSchema,
  latestBarSortSchema,
  viewModeSchema,
  type BarOrientation,
  type BinType,
  type ChartType,
  type LatestBarSort,
  type PackageGroup,
  type ShowDataMode,
  type TimeRange,
  type TransformMode,
  type ViewMode,
} from '~/components/npm-stats/shared'

const LazyNPMStatsChart = React.lazy(() =>
  import('~/components/npm-stats/NPMStatsChart').then((m) => ({
    default: m.NPMStatsChart,
  })),
)

function ChartFallback({ height }: { height: number }) {
  return (
    <div
      className="animate-pulse rounded bg-gray-100 dark:bg-gray-800/40"
      style={{ height }}
    />
  )
}

const transformModeSchema = v.picklist(['none', 'normalize-y'])
const binTypeSchema = v.picklist(['yearly', 'monthly', 'weekly', 'daily'])
const showDataModeSchema = v.picklist(['all', 'complete'])
const bucketOffsetSchema = v.pipe(
  v.number(),
  v.integer(),
  v.minValue(-5000),
  v.maxValue(0),
)
const timelineZoomTimeSchema = v.pipe(v.number(), v.integer(), v.minValue(0))
export const npmStatsSearchSchema = v.object({
  packageGroups: v.fallback(
    v.optional(packageGroupsSchema, defaultPackageGroups),
    defaultPackageGroups,
  ),
  range: v.fallback(
    v.optional(
      v.picklist([
        '7-days',
        '30-days',
        '90-days',
        '180-days',
        '365-days',
        '730-days',
        '1825-days',
        'all-time',
      ]),
      '1825-days',
    ),
    '1825-days',
  ),
  transform: v.fallback(v.optional(transformModeSchema, 'none'), 'none'),
  binType: v.fallback(v.optional(binTypeSchema, 'weekly'), 'weekly'),
  viewMode: v.fallback(v.optional(viewModeSchema, 'history'), 'history'),
  chartType: v.fallback(v.optional(chartTypeSchema, 'line'), 'line'),
  barOrientation: v.fallback(v.optional(barOrientationSchema), undefined),
  barSort: v.fallback(v.optional(latestBarSortSchema, 'value'), 'value'),
  bucketOffset: v.fallback(v.optional(bucketOffsetSchema, 0), 0),
  playbackIntervalMs: v.fallback(
    v.optional(playbackIntervalMsSchema, defaultPlaybackIntervalMs),
    defaultPlaybackIntervalMs,
  ),
  playbackLoop: v.fallback(v.optional(v.boolean(), false), false),
  playbackPlaying: v.fallback(v.optional(v.boolean(), false), false),
  showDataMode: v.fallback(v.optional(showDataModeSchema, 'all'), 'all'),
  normalizeBaseline: v.fallback(v.optional(v.boolean(), true), true),
  showBaseline: v.fallback(v.optional(v.boolean(), false), false),
  showLegend: v.fallback(v.optional(v.boolean(), false), false),
  height: v.fallback(v.optional(chartHeightSchema, 400), 400),
  width: v.fallback(v.optional(chartWidthSchema), undefined),
  timelineStart: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
  timelineEnd: v.fallback(v.optional(timelineZoomTimeSchema), undefined),
})

export const defaultNpmStatsSearch = v.parse(npmStatsSearchSchema, {})

export const Route = createFileRoute('/stats/npm/')({
  validateSearch: npmStatsSearchSchema,
  search: { middlewares: [stripSearchParams(defaultNpmStatsSearch)] },
  loaderDeps: ({ search }) => ({
    packageList: search.packageGroups?.map(getPackageGroupLabel).join(' vs '),
    packageNames: search.packageGroups?.map(getPackageGroupLabel) ?? [],
  }),
  loader: async ({ deps }) => {
    return deps
  },
  head: ({ loaderData }) => {
    const packageList = loaderData?.packageList ?? ''
    const packageNames = loaderData?.packageNames ?? []
    const hasPackages = packageNames.length > 0

    // Create SEO-optimized title - lead with the comparison for better CTR
    const title = hasPackages
      ? `${packageList} - NPM Download Stats & Trends | Compare Packages`
      : 'NPM Download Stats & Trends - Compare Package Popularity | TanStack'

    // Dynamic description based on packages being compared
    const description = hasPackages
      ? `Compare ${packageList} npm downloads side-by-side. View download trends, weekly stats, and historical data. Free npm package comparison tool - faster than npmtrends.`
      : 'Compare npm package downloads with interactive charts. Track download trends, analyze package popularity, and make informed decisions. Free alternative to npmtrends and npm-stat.'

    // Keywords for better discoverability
    const keywords = hasPackages
      ? `${packageNames.join(', ')}, npm downloads, npm stats, package comparison, npm trends, download statistics`
      : 'npm downloads, npm statistics, npm trends, compare npm packages, package popularity, npm download stats, javascript packages'

    // JSON-LD structured data for better search appearance
    const jsonLd = {
      '@context': 'https://schema.org',
      '@type': 'WebApplication',
      name: 'TanStack NPM Stats',
      description:
        'Compare npm package downloads with interactive charts and historical data',
      url: 'https://tanstack.com/stats/npm',
      applicationCategory: 'DeveloperApplication',
      operatingSystem: 'Web',
      offers: {
        '@type': 'Offer',
        price: '0',
        priceCurrency: 'USD',
      },
      creator: {
        '@type': 'Organization',
        name: 'TanStack',
        url: 'https://tanstack.com',
      },
    }

    return {
      meta: seo({
        title,
        description,
        keywords,
      }),
      scripts: [
        {
          type: 'application/ld+json',
          children: JSON.stringify(jsonLd),
        },
      ],
    }
  },
  component: RouteComponent,
})

type _NpmStatsSearch = {
  packageGroups?: Array<{
    name?: string
    label?: string
    hidden?: boolean
    color?: string
    baseline?: boolean
    baselineLabel?: string
    packages: Array<{ name?: string; hidden?: boolean }>
  }>
  range?: TimeRange
  transform?: 'none' | 'normalize-y'
  binType?: BinType
  viewMode?: ViewMode
  chartType?: ChartType
  barOrientation?: BarOrientation
  barSort?: LatestBarSort
  bucketOffset?: number
  playbackIntervalMs?: number
  playbackLoop?: boolean
  playbackPlaying?: boolean
  showDataMode?: 'all' | 'complete'
  showLegend?: boolean
  height?: number
  width?: number
  timelineStart?: number
  timelineEnd?: number
}

function RouteComponent() {
  const search = Route.useSearch()
  const packageGroups: PackageGroup[] =
    search.packageGroups ?? defaultPackageGroups
  const range: TimeRange = search.range ?? '1825-days'
  const transform: TransformMode = search.transform ?? 'none'
  const binTypeParam: BinType | undefined = search.binType
  const viewMode: ViewMode = search.viewMode ?? 'history'
  const chartTypeParam: ChartType = search.chartType ?? 'line'
  const barOrientationParam: BarOrientation =
    search.barOrientation ??
    (chartTypeParam === 'horizontal-bar' ? 'horizontal' : 'vertical')
  const barSort: LatestBarSort = search.barSort ?? 'value'
  const showDataModeParam: ShowDataMode = search.showDataMode ?? 'all'
  const normalizeBaseline: boolean = search.normalizeBaseline ?? true
  const showBaseline: boolean = search.showBaseline ?? false
  const showLegend: boolean = search.showLegend ?? false
  const bucketOffsetParam: number = search.bucketOffset ?? 0
  const playbackIntervalMs: number =
    search.playbackIntervalMs ?? defaultPlaybackIntervalMs
  const playbackLoop: boolean = search.playbackLoop ?? false
  const playbackPlaying: boolean = search.playbackPlaying ?? false
  const height: number = search.height ?? 400
  const width: number | undefined = search.width
  const timelineStart: number | undefined = search.timelineStart
  const timelineEnd: number | undefined = search.timelineEnd
  const [combiningPackage, setCombiningPackage] = React.useState<string | null>(
    null,
  )
  const navigate = Route.useNavigate()
  const [colorPickerPackage, setColorPickerPackage] = React.useState<
    string | null
  >(null)
  const [colorPickerPosition, setColorPickerPosition] = React.useState<{
    x: number
    y: number
  } | null>(null)
  const [openMenuPackage, setOpenMenuPackage] = React.useState<string | null>(
    null,
  )
  const [chartActionsContainer, setChartActionsContainer] =
    React.useState<HTMLDivElement | null>(null)
  const [chartActionsMounted, setChartActionsMounted] = React.useState(false)

  const binType: BinType = binTypeParam ?? defaultRangeBinTypes[range]
  const chartType = isChartTypeValidForViewMode(viewMode, chartTypeParam)
    ? chartTypeParam
    : getDefaultChartTypeForViewMode(viewMode)
  const barOrientation: BarOrientation =
    viewMode === 'latest' &&
    (chartType === 'bar' || chartType === 'stacked-bar')
      ? barOrientationParam
      : 'vertical'
  const latestBucketBounds = getLatestBucketOffsetBounds({ binType, range })
  const bucketOffset = Math.max(
    latestBucketBounds.minOffset,
    Math.min(bucketOffsetParam, latestBucketBounds.maxOffset),
  )
  const queryPackageGroupEntries = packageGroups
    .map((packageGroup, index) => ({ packageGroup, index }))
    .filter(
      ({ packageGroup }) => viewMode === 'history' || !packageGroup.baseline,
    )
  const queryPackageGroups = queryPackageGroupEntries.map(
    ({ packageGroup }) => packageGroup,
  )
  const queryPackageGroupIndexes = queryPackageGroupEntries.map(
    ({ index }) => index,
  )
  const tableTransform: TransformMode =
    viewMode === 'history' && chartType === 'line' ? transform : 'none'
  const canToggleShowBaseline = viewMode === 'history' && chartType === 'line'

  const handleBinnedChange = (value: BinType) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        binType: value,
        bucketOffset: 0,
        playbackPlaying: false,
        timelineStart: undefined,
        timelineEnd: undefined,
      }),
      resetScroll: false,
    })
  }

  const handleViewModeChange = (mode: ViewMode) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        viewMode: mode,
        chartType: isChartTypeValidForViewMode(mode, chartType)
          ? chartType
          : getDefaultChartTypeForViewMode(mode),
        transform: mode === 'latest' ? 'none' : prev.transform,
        barOrientation,
        showBaseline: mode === 'latest' ? false : prev.showBaseline,
        playbackPlaying: mode === 'latest' ? prev.playbackPlaying : false,
        bucketOffset: 0,
        timelineStart: undefined,
        timelineEnd: undefined,
      }),
      resetScroll: false,
    })
  }

  const handleChartTypeChange = (nextChartType: ChartType) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        chartType: nextChartType,
        barOrientation,
        transform: nextChartType === 'line' ? prev.transform : 'none',
      }),
      resetScroll: false,
    })
  }

  const handleBarSortChange = (nextBarSort: LatestBarSort) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        barSort: nextBarSort,
      }),
      resetScroll: false,
    })
  }

  const handleBarOrientationChange = (nextBarOrientation: BarOrientation) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        barOrientation: nextBarOrientation,
      }),
      resetScroll: false,
    })
  }

  const handleBucketOffsetChange = (nextBucketOffset: number) => {
    const nextOffset = Math.max(
      latestBucketBounds.minOffset,
      Math.min(nextBucketOffset, latestBucketBounds.maxOffset),
    )

    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        bucketOffset: nextOffset,
      }),
      replace: true,
      resetScroll: false,
    })
  }

  const handlePlaybackIntervalChange = (nextPlaybackIntervalMs: number) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        playbackIntervalMs: nextPlaybackIntervalMs,
      }),
      replace: true,
      resetScroll: false,
    })
  }

  const handlePlaybackLoopChange = (nextPlaybackLoop: boolean) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        playbackLoop: nextPlaybackLoop,
      }),
      resetScroll: false,
    })
  }

  const handlePlaybackPlayingChange = (nextPlaybackPlaying: boolean) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        playbackPlaying: nextPlaybackPlaying,
      }),
      replace: true,
      resetScroll: false,
    })
  }

  const handleAddBaseline = (packageName: string, color?: string) => {
    navigate({
      to: '.',
      search: (prev) => {
        const groups = prev.packageGroups ?? []
        const alreadyBaseline = groups.some(
          (pg) =>
            pg.baseline && pg.packages.some((p) => p.name === packageName),
        )
        if (alreadyBaseline) return prev
        return {
          ...prev,
          packageGroups: [
            ...groups,
            {
              packages: [{ name: packageName, hidden: true }],
              color,
              baseline: true,
            },
          ],
        }
      },
      resetScroll: false,
    })
  }

  const handleApplyBaselinePreset = (preset: BaselinePreset) => {
    navigate({
      to: '.',
      search: (prev) => {
        const nonBaseline = (prev.packageGroups ?? []).filter(
          (pg) => !pg.baseline,
        )
        return {
          ...prev,
          bucketOffset: 0,
          playbackPlaying: false,
          packageGroups: [
            ...nonBaseline,
            ...preset.packages.map((p) => ({
              packages: [{ name: p.name, hidden: true }],
              color: p.color,
              baseline: true,
              baselineLabel: preset.title,
            })),
          ],
        }
      },
      resetScroll: false,
    })
  }

  const handleClearBaselines = () => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        packageGroups: prev.packageGroups?.filter((pg) => !pg.baseline),
        showBaseline: false,
      }),
      resetScroll: false,
    })
  }

  const handleToggleShowBaseline = () => {
    if (!canToggleShowBaseline) return

    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        showBaseline: !prev.showBaseline,
      }),
      resetScroll: false,
    })
  }

  const handleToggleNormalizeBaseline = () => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        normalizeBaseline: !(prev.normalizeBaseline ?? true),
      }),
      resetScroll: false,
    })
  }

  const handleShowLegendChange = (nextShowLegend: boolean) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        showLegend: nextShowLegend,
      }),
      replace: true,
      resetScroll: false,
    })
  }

  const handleShowDataModeChange = (mode: ShowDataMode) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        showDataMode: mode,
      }),
      resetScroll: false,
    })
  }

  const togglePackageVisibility = (index: number, packageName: string) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        packageGroups: prev.packageGroups?.map((pkg, i) =>
          i === index
            ? pkg.label === packageName
              ? { ...pkg, hidden: !pkg.hidden }
              : {
                  ...pkg,
                  packages: pkg.packages.map((p) =>
                    p.name === packageName ? { ...p, hidden: !p.hidden } : p,
                  ),
                }
            : pkg,
        ),
      }),
      replace: true,
      resetScroll: false,
    })
  }

  const npmQuery = useQuery(
    npmQueryOptions({
      packageGroups: queryPackageGroups,
      range,
      viewMode,
      binType,
      bucketOffset,
    }),
  )
  const displayQueryData = React.useMemo(
    () =>
      viewMode === 'latest'
        ? selectLatestBucketQueryData({
            queryData: npmQuery.data,
            binType,
            bucketOffset,
          })
        : npmQuery.data,
    [binType, bucketOffset, npmQuery.data, viewMode],
  )
  const packagePillQueryData = (() => {
    if (viewMode === 'history') return npmQuery.data

    let queryIndex = 0
    return packageGroups.map((packageGroup) => {
      if (packageGroup.baseline) return undefined

      const queryData = npmQuery.data?.[queryIndex]
      queryIndex++
      return queryData
    })
  })()

  const handleRemoveFromGroup = (mainPackage: string, subPackage: string) => {
    // Find the package group
    const packageGroup = packageGroups?.find((pkg) =>
      pkg.packages.some((p) => p.name === mainPackage),
    )
    if (!packageGroup) return

    // Remove the subpackage
    const updatedPackages = packageGroup.packages.filter(
      (p) => p.name !== subPackage,
    )

    // Update the packages array
    const newPackages = (packageGroups ?? [])
      .map((pkg) =>
        pkg === packageGroup ? { ...pkg, packages: updatedPackages } : pkg,
      )
      .filter((pkg) => pkg.packages.length > 0)

    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        packageGroups: newPackages,
      }),
      resetScroll: false,
    })
  }

  const handleRemovePackageName = (packageGroupIndex: number) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        packageGroups: prev.packageGroups?.filter(
          (_: unknown, i: number) => i !== packageGroupIndex,
        ),
      }),
      resetScroll: false,
    })
  }

  const handleLabelChange = (packageGroupIndex: number, label: string) => {
    navigate({
      to: '.',
      search: (prev) => {
        const nextLabel = label.slice(0, 80)
        const hasNextLabel = nextLabel.trim().length > 0
        const groups = prev.packageGroups ?? packageGroups

        return {
          ...prev,
          packageGroups: groups.map((pkg, i) => {
            if (i !== packageGroupIndex) return pkg

            const nextPackageGroup = { ...pkg }
            if (hasNextLabel) {
              nextPackageGroup.label = nextLabel
            } else {
              delete nextPackageGroup.label
              delete nextPackageGroup.hidden
            }

            return nextPackageGroup
          }),
        }
      },
      replace: true,
      resetScroll: false,
    })
  }

  const handleRangeChange = (newRange: TimeRange) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        range: newRange,
        binType: defaultRangeBinTypes[newRange],
        bucketOffset: 0,
        playbackPlaying: false,
        timelineStart: undefined,
        timelineEnd: undefined,
      }),
      resetScroll: false,
    })
  }

  const handleTransformChange = (mode: TransformMode) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        transform: mode,
      }),
      resetScroll: false,
    })
  }

  const handleCombinePackage = (packageName: string) => {
    setCombiningPackage(packageName)
  }

  const handleColorClick = (packageName: string, event: React.MouseEvent) => {
    const rect = event.currentTarget.getBoundingClientRect()
    setColorPickerPosition({ x: rect.left, y: rect.bottom + 5 })
    setColorPickerPackage(packageName)
  }

  const handleColorChange = useThrottledCallback(
    (packageName: string, color: string | null) => {
      navigate({
        to: '.',
        search: (prev) => {
          const packageGroup = packageGroups.find(
            (pkg) =>
              pkg.label === packageName ||
              pkg.packages.some((p) => p.name === packageName),
          )
          if (!packageGroup) return prev

          const newPackages = packageGroups.map((pkg) =>
            pkg === packageGroup
              ? color === null
                ? { packages: pkg.packages }
                : { ...pkg, color }
              : pkg,
          )

          return {
            ...prev,
            packageGroups: newPackages,
          }
        },
        replace: true,
        resetScroll: false,
      })
    },
    {
      wait: 100,
    },
  )

  const navigateChartSize = React.useCallback(
    (size: ResizableSizeChange, replace: boolean) => {
      navigate({
        to: '.',
        search: (prev) => ({
          ...prev,
          ...('height' in size ? { height: size.height } : {}),
          ...('width' in size ? { width: size.width } : {}),
        }),
        replace,
        resetScroll: false,
      })
    },
    [navigate],
  )

  const chartSizeThrottler = useThrottler(
    (size: ResizableSizeChange) => navigateChartSize(size, true),
    {
      wait: 16,
    },
  )

  const onSizeChange = React.useCallback(
    (size: ResizableSizeChange, options: ResizeChangeOptions) => {
      if (options.replace) {
        chartSizeThrottler.maybeExecute(size)
        return
      }

      chartSizeThrottler.cancel()
      if (!('height' in size) && !('width' in size)) return

      navigateChartSize(size, false)
    },
    [chartSizeThrottler, navigateChartSize],
  )

  const handleTimelineRangeChange = React.useCallback(
    (nextRange: { end: number | undefined; start: number | undefined }) => {
      navigate({
        to: '.',
        search: (prev) => ({
          ...prev,
          timelineStart: nextRange.start,
          timelineEnd: nextRange.end,
        }),
        replace: true,
        resetScroll: false,
      })
    },
    [navigate],
  )

  const buildEmbedUrl = React.useCallback(
    (options: NpmStatsChartEmbedOptions) => {
      const params = new URLSearchParams()

      params.set('packageGroups', JSON.stringify(packageGroups))
      params.set('range', range)
      params.set('transform', transform)
      params.set('binType', binType)
      params.set('viewMode', viewMode)
      params.set('chartType', chartType)
      params.set('barOrientation', barOrientation)
      params.set('barSort', barSort)
      params.set('bucketOffset', `${bucketOffset}`)
      params.set('playbackIntervalMs', `${playbackIntervalMs}`)
      params.set('showDataMode', showDataModeParam)
      params.set('normalizeBaseline', `${normalizeBaseline}`)
      params.set('showBaseline', `${viewMode === 'history'}`)
      params.set('showLegend', `${options.showLegend}`)

      if (
        options.includeTimelineRange &&
        timelineStart !== undefined &&
        timelineEnd !== undefined
      ) {
        params.set('timelineStart', `${timelineStart}`)
        params.set('timelineEnd', `${timelineEnd}`)
      }

      if (options.lockWidth && width !== undefined) {
        params.set('width', `${width}`)
      }

      return `/stats/npm/embed?${params}`
    },
    [
      barOrientation,
      barSort,
      binType,
      bucketOffset,
      chartType,
      normalizeBaseline,
      packageGroups,
      playbackIntervalMs,
      range,
      showDataModeParam,
      timelineEnd,
      timelineStart,
      transform,
      viewMode,
      width,
    ],
  )

  const embedConfig = React.useMemo(
    () => ({
      buildUrl: buildEmbedUrl,
      hasTimelineRange:
        timelineStart !== undefined && timelineEnd !== undefined,
      hasWidth: width !== undefined,
    }),
    [buildEmbedUrl, timelineEnd, timelineStart, width],
  )

  const handleMenuOpenChange = (packageName: string, open: boolean) => {
    if (!open) {
      setOpenMenuPackage(null)
    } else {
      setOpenMenuPackage(packageName)
    }
  }

  const handleAddPackage = (packageName: string) => {
    navigate({
      to: '.',
      search: (prev) => ({
        ...prev,
        packageGroups: [
          ...(prev.packageGroups ?? []),
          {
            packages: [{ name: packageName }],
          },
        ],
      }),
      resetScroll: false,
    })
  }

  const handleAddToGroup = (packageName: string) => {
    if (!combiningPackage) return

    // Find the package group that contains the combining package
    const packageGroup = packageGroups.find((pkg) =>
      pkg.packages.some((p) => p.name === combiningPackage),
    )

    if (packageGroup) {
      // Update existing package group
      const newPackages = packageGroups.map((pkg) =>
        pkg === packageGroup
          ? {
              ...pkg,
              packages: [...pkg.packages, { name: packageName }],
            }
          : pkg,
      )

      navigate({
        to: '.',
        search: (prev) => ({
          ...prev,
          packageGroups: newPackages,
        }),
        resetScroll: false,
      })
    } else {
      // Create new package group
      navigate({
        to: '.',
        search: (prev) => ({
          ...prev,
          packageGroups: [
            ...packageGroups,
            {
              packages: [{ name: combiningPackage }, { name: packageName }],
            },
          ],
        }),
        resetScroll: false,
      })
    }

    setCombiningPackage(null)
  }

  // Generate dynamic H1 based on packages being compared
  const packageListForH1 = packageGroups.map(getPackageGroupLabel).join(' vs ')

  return (
    <div className="min-h-dvh p-2 sm:p-4 space-y-2 sm:space-y-4">
      <header className="sr-only">
        <h1>
          NPM Download Comparison
          {packageGroups.length > 0 ? `: ${packageListForH1}` : ''}
        </h1>
        <p>
          Compare npm package downloads with interactive charts. Track trends
          and make data-driven decisions.
        </p>
      </header>

      <div className="flex gap-4">
        <Card className="flex-1 space-y-4 p-4 max-w-full">
          <div className="flex flex-wrap items-center gap-1">
            <PackageSearch onSelect={handleAddPackage} />
            <ChartControls
              viewMode={viewMode}
              chartType={chartType}
              range={range}
              binType={binType}
              transform={transform}
              showDataMode={showDataModeParam}
              barSort={barSort}
              barOrientation={barOrientation}
              onViewModeChange={handleViewModeChange}
              onChartTypeChange={handleChartTypeChange}
              onRangeChange={handleRangeChange}
              onBinTypeChange={handleBinnedChange}
              onTransformChange={handleTransformChange}
              onShowDataModeChange={handleShowDataModeChange}
              onBarSortChange={handleBarSortChange}
              onBarOrientationChange={handleBarOrientationChange}
            />
            <Tooltip
              content="Compare npm package downloads with interactive charts. Track trends and make data-driven decisions."
              placement="bottom"
            >
              <button
                aria-label="About NPM download comparison"
                className="flex size-6 items-center justify-center rounded bg-gray-500/10 text-gray-500 hover:bg-gray-500/20 hover:text-blue-500"
                type="button"
              >
                <QuestionIcon className="size-3" />
              </button>
            </Tooltip>
          </div>
          <PackagePills
            packageGroups={packageGroups}
            queryData={packagePillQueryData}
            onColorClick={handleColorClick}
            onToggleVisibility={togglePackageVisibility}
            onRemove={handleRemovePackageName}
            onCombinePackage={handleCombinePackage}
            onLabelChange={handleLabelChange}
            onRemoveFromGroup={handleRemoveFromGroup}
            openMenuPackage={openMenuPackage}
            onMenuOpenChange={handleMenuOpenChange}
          />

          <div
            className={
              viewMode === 'history'
                ? 'flex flex-wrap items-center justify-between gap-2'
                : 'flex flex-wrap items-center gap-2'
            }
          >
            {viewMode === 'history' ? (
              <div className="min-w-0 flex-1">
                <BaselineSection
                  packageGroups={packageGroups}
                  presets={getBaselinePresets()}
                  canToggleShowBaseline={canToggleShowBaseline}
                  normalizeBaseline={normalizeBaseline}
                  showBaseline={showBaseline}
                  onToggleNormalizeBaseline={handleToggleNormalizeBaseline}
                  onToggleShowBaseline={handleToggleShowBaseline}
                  onAddBaseline={handleAddBaseline}
                  onApplyPreset={handleApplyBaselinePreset}
                  onClearBaselines={handleClearBaselines}
                />
              </div>
            ) : (
              <LatestBucketNavigator
                binType={binType}
                range={range}
                bucketOffset={bucketOffset}
                isLooping={playbackLoop}
                isPlaying={playbackPlaying}
                playbackIntervalMs={playbackIntervalMs}
                onBucketOffsetChange={handleBucketOffsetChange}
                onLoopingChange={handlePlaybackLoopChange}
                onPlaybackIntervalChange={handlePlaybackIntervalChange}
                onPlayingChange={handlePlaybackPlayingChange}
              />
            )}
            <div
              className="ml-auto flex min-h-6 items-center justify-end"
              ref={setChartActionsContainer}
            >
              {chartActionsMounted ? null : <DisabledChartActions />}
            </div>
          </div>

          {/* Combine Package Dialog */}
          <Dialog
            open={combiningPackage !== null}
            onOpenChange={(open) => {
              if (!open) setCombiningPackage(null)
            }}
          >
            <DialogContent size="md">
              <DialogHeader
                title={`Add packages to ${combiningPackage ?? ''}`}
              />
              <DialogBody className="pb-6">
                {combiningPackage && (
                  <PackageSearch
                    onSelect={handleAddToGroup}
                    placeholder="Search for packages to add..."
                    // eslint-disable-next-line jsx-a11y/no-autofocus
                    autoFocus={true}
                  />
                )}
              </DialogBody>
            </DialogContent>
          </Dialog>

          {/* Color Picker Popover */}
          {colorPickerPackage && colorPickerPosition && (
            <ColorPickerPopover
              packageName={colorPickerPackage}
              position={colorPickerPosition}
              currentColor={getPackageColor(colorPickerPackage, packageGroups)}
              onColorChange={handleColorChange}
              onReset={(pkgName) => handleColorChange(pkgName, null)}
              onClose={() => {
                setColorPickerPackage(null)
                setColorPickerPosition(null)
              }}
            />
          )}

          {queryPackageGroups.length ? (
            <div className="">
              <div className="space-y-2 sm:space-y-4">
                <div className="relative">
                  {npmQuery.isFetching && npmQuery.data ? (
                    <div className="absolute inset-0 flex items-center justify-center bg-white/30 dark:bg-black/30 z-10 rounded-lg">
                      <div className="flex flex-col items-center gap-4">
                        <Spinner />
                        <div className="text-sm font-medium text-gray-900 dark:text-gray-100">
                          Updating...
                        </div>
                      </div>
                    </div>
                  ) : null}
                  <Resizable
                    height={height}
                    width={width}
                    onSizeChange={onSizeChange}
                  >
                    {npmQuery.isLoading ? (
                      <div
                        className="flex items-center justify-center"
                        style={{ height }}
                      >
                        <div className="flex flex-col items-center gap-4">
                          <Spinner />
                          <div className="text-sm text-gray-600 dark:text-gray-400">
                            Loading download statistics...
                          </div>
                        </div>
                      </div>
                    ) : (
                      <React.Suspense
                        fallback={<ChartFallback height={height} />}
                      >
                        <LazyNPMStatsChart
                          actionsContainer={chartActionsContainer}
                          embedConfig={embedConfig}
                          onActionsMountedChange={setChartActionsMounted}
                          range={range}
                          queryData={displayQueryData}
                          height={height}
                          viewMode={viewMode}
                          chartType={chartType}
                          barSort={barSort}
                          barOrientation={barOrientation}
                          transform={transform}
                          binType={binType}
                          packages={queryPackageGroups}
                          showDataMode={showDataModeParam}
                          normalizeBaseline={
                            viewMode === 'history' && normalizeBaseline
                          }
                          showBaseline={viewMode === 'history' && showBaseline}
                          showLegend={showLegend}
                          timelineStart={timelineStart}
                          timelineEnd={timelineEnd}
                          animationQueryData={npmQuery.data}
                          animationBucketOffsetBounds={latestBucketBounds}
                          animationFrameIntervalMs={playbackIntervalMs}
                          onShowLegendChange={handleShowLegendChange}
                          onTimelineRangeChange={handleTimelineRangeChange}
                        />
                      </React.Suspense>
                    )}
                  </Resizable>
                </div>
                <StatsTable
                  queryData={displayQueryData}
                  packageGroups={queryPackageGroups}
                  packageGroupIndexes={queryPackageGroupIndexes}
                  binType={binType}
                  transform={tableTransform}
                  viewMode={viewMode}
                  onColorClick={handleColorClick}
                  onToggleVisibility={togglePackageVisibility}
                  onRemove={handleRemovePackageName}
                />
              </div>
            </div>
          ) : null}

          {/* Popular Comparisons Section */}
          <PopularComparisons comparisons={getPopularComparisons()} />
        </Card>
      </div>

      {/* FAQ Section for SEO */}
      <section className="max-w-4xl mt-8 space-y-6">
        <h2 className="text-xl font-semibold text-gray-900 dark:text-gray-100">
          Frequently Asked Questions
        </h2>
        <div className="space-y-4">
          <details className="group bg-gray-500/5 rounded-lg p-4">
            <summary className="font-medium cursor-pointer list-none flex justify-between items-center">
              How are npm download statistics calculated?
              <span className="text-gray-400 group-open:rotate-180 transition-transform">
                ▼
              </span>
            </summary>
            <p className="mt-3 text-sm text-gray-600 dark:text-gray-400">
              NPM download statistics are sourced from the official npm registry
              API. Downloads are counted each time a package is installed via
              npm, yarn, or pnpm. These numbers include downloads from CI/CD
              pipelines, development machines, and production deployments.
            </p>
          </details>

          <details className="group bg-gray-500/5 rounded-lg p-4">
            <summary className="font-medium cursor-pointer list-none flex justify-between items-center">
              What's the difference between weekly and daily downloads?
              <span className="text-gray-400 group-open:rotate-180 transition-transform">
                ▼
              </span>
            </summary>
            <p className="mt-3 text-sm text-gray-600 dark:text-gray-400">
              Daily downloads show the exact number of downloads per day, useful
              for spotting short-term trends and anomalies. Weekly downloads
              aggregate 7 days of data, smoothing out day-to-day variations and
              making it easier to identify long-term growth patterns.
            </p>
          </details>

          <details className="group bg-gray-500/5 rounded-lg p-4">
            <summary className="font-medium cursor-pointer list-none flex justify-between items-center">
              How does TanStack NPM Stats compare to npmtrends?
              <span className="text-gray-400 group-open:rotate-180 transition-transform">
                ▼
              </span>
            </summary>
            <p className="mt-3 text-sm text-gray-600 dark:text-gray-400">
              TanStack NPM Stats offers faster load times with cached data,
              flexible time ranges up to all-time history, advanced features
              like baseline comparisons and relative growth charts, plus the
              ability to combine multiple packages into a single trend line for
              tracking package migrations (e.g., react-query to
              @tanstack/react-query).
            </p>
          </details>

          <details className="group bg-gray-500/5 rounded-lg p-4">
            <summary className="font-medium cursor-pointer list-none flex justify-between items-center">
              Why do some packages show zero downloads before a certain date?
              <span className="text-gray-400 group-open:rotate-180 transition-transform">
                ▼
              </span>
            </summary>
            <p className="mt-3 text-sm text-gray-600 dark:text-gray-400">
              NPM download statistics are only available from January 10, 2015
              onwards. Additionally, packages will show zero downloads before
              their initial publish date. Some packages may also be renamed or
              scoped (e.g., react-query became @tanstack/react-query), so
              historical data may appear under different package names.
            </p>
          </details>

          <details className="group bg-gray-500/5 rounded-lg p-4">
            <summary className="font-medium cursor-pointer list-none flex justify-between items-center">
              Can I compare any npm package?
              <span className="text-gray-400 group-open:rotate-180 transition-transform">
                ▼
              </span>
            </summary>
            <p className="mt-3 text-sm text-gray-600 dark:text-gray-400">
              Yes! You can compare any public package on the npm registry.
              Simply search for packages by name and add them to your
              comparison. You can compare multiple packages simultaneously and
              even group related packages together (like combining legacy and
              new package names) for accurate historical tracking.
            </p>
          </details>
        </div>
      </section>
    </div>
  )
}
````

## File: src/routes/stats/index.tsx
````typescript
import { redirect, createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/stats/')({
  beforeLoad: () => {
    throw redirect({ to: '/stats/npm' })
  },
})
````
