This file is a merged representation of the entire codebase, combined into a single document by Repomix.

# File Summary

## Purpose
This file contains a packed representation of the entire repository's contents.
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
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)

# Directory Structure
````
.claude/
  CLAUDE.md
  settings.json
.cursor/
  rules/
    ultracite.mdc
  hooks.json
actions/
  package/
    get.ts
    search.ts
app/
  [package]/
    components/
      content.tsx
      footer.tsx
      header.tsx
    page.tsx
  og/
    route.tsx
  svg/
    route.ts
  globals.css
  layout.tsx
  page.tsx
components/
  ui/
    accordion.tsx
    alert-dialog.tsx
    alert.tsx
    aspect-ratio.tsx
    avatar.tsx
    badge.tsx
    breadcrumb.tsx
    button-group.tsx
    button.tsx
    calendar.tsx
    card.tsx
    carousel.tsx
    chart.tsx
    checkbox.tsx
    collapsible.tsx
    command.tsx
    context-menu.tsx
    dialog.tsx
    drawer.tsx
    dropdown-menu.tsx
    empty.tsx
    field.tsx
    form.tsx
    hover-card.tsx
    input-group.tsx
    input-otp.tsx
    input.tsx
    item.tsx
    kbd.tsx
    label.tsx
    menubar.tsx
    navigation-menu.tsx
    pagination.tsx
    popover.tsx
    progress.tsx
    radio-group.tsx
    resizable.tsx
    scroll-area.tsx
    select.tsx
    separator.tsx
    sheet.tsx
    sidebar.tsx
    skeleton.tsx
    slider.tsx
    sonner.tsx
    spinner.tsx
    switch.tsx
    table.tsx
    tabs.tsx
    textarea.tsx
    toggle-group.tsx
    toggle.tsx
    tooltip.tsx
  chart-loading.tsx
  chart.tsx
  download-tooltip.tsx
  embed-dialog.tsx
  empty-state.tsx
  footer.tsx
  github.tsx
  grouping-selector.tsx
  header.tsx
  logo.tsx
  main.tsx
  metric-selector.tsx
  package-search.tsx
  screenshot.tsx
  theme-toggle.tsx
  time-range-selector.tsx
  zero-mode-toggle.tsx
hooks/
  use-mobile.ts
lib/
  chart-utils.ts
  download-data.ts
  utils.ts
providers/
  filters.tsx
tests/
  fixtures/
    download-gaps.json
  download-data.test.mjs
  serve-fixture.mjs
  verify-preview.mjs
.gitattributes
.gitignore
biome.jsonc
components.json
next.config.ts
package.json
postcss.config.mjs
README.md
tsconfig.json
tsconfig.test.json
````

# Files

## File: .claude/CLAUDE.md
````markdown
# Ultracite Code Standards

This project uses **Ultracite**, a zero-config Biome preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `npx ultracite fix`
- **Check for issues**: `npx ultracite check`
- **Diagnose setup**: `npx ultracite doctor`

Biome (the underlying engine) provides extremely fast Rust-based linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Framework-Specific Guidance

**Next.js:**
- Use Next.js `<Image>` component for images
- Use `next/head` or App Router metadata API for head elements
- Use Server Components for async data fetching instead of async Client Components

**React 19+:**
- Use ref as a prop instead of `React.forwardRef`

**Solid/Svelte/Vue/Qwik:**
- Use `class` and `for` attributes (not `className` or `htmlFor`)

---

## Testing

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

## When Biome Can't Help

Biome's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

Most formatting and common issues are automatically fixed by Biome. Run `npx ultracite fix` before committing to ensure compliance.
````

## File: .claude/settings.json
````json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx ultracite fix"
          }
        ]
      }
    ]
  }
}
````

## File: .cursor/rules/ultracite.mdc
````
---
description: Ultracite Rules - AI-Ready Formatter and Linter
globs: "**/*.{ts,tsx,js,jsx,json,jsonc,html,vue,svelte,astro,css,yaml,yml,graphql,gql,md,mdx,grit}"
alwaysApply: false
---

# Ultracite Code Standards

This project uses **Ultracite**, a zero-config Biome preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `npx ultracite fix`
- **Check for issues**: `npx ultracite check`
- **Diagnose setup**: `npx ultracite doctor`

Biome (the underlying engine) provides extremely fast Rust-based linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Framework-Specific Guidance

**Next.js:**
- Use Next.js `<Image>` component for images
- Use `next/head` or App Router metadata API for head elements
- Use Server Components for async data fetching instead of async Client Components

**React 19+:**
- Use ref as a prop instead of `React.forwardRef`

**Solid/Svelte/Vue/Qwik:**
- Use `class` and `for` attributes (not `className` or `htmlFor`)

---

## Testing

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

## When Biome Can't Help

Biome's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

Most formatting and common issues are automatically fixed by Biome. Run `npx ultracite fix` before committing to ensure compliance.
````

## File: .cursor/hooks.json
````json
{
  "version": 1,
  "hooks": {
    "afterFileEdit": [
      {
        "command": "npx ultracite fix"
      }
    ]
  }
}
````

## File: actions/package/get.ts
````typescript
"use server";

import { Redis } from "@upstash/redis";
import { after } from "next/server";

const redis = Redis.fromEnv();
const isDevelopment = process.env.NODE_ENV === "development";

export interface PackageData {
  start: string;
  end: string;
  package: string;
  downloads: {
    downloads: number;
    day: string;
  }[];
}

const formatDateForAPI = (date: Date): string => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");

  return `${year}-${month}-${day}`;
};

const convertTimeRangeToAPIFormat = (timeRange: string): string => {
  // Native npm API ranges - pass through as-is
  if (
    timeRange === "last-week" ||
    timeRange === "last-month" ||
    timeRange === "last-year"
  ) {
    return timeRange;
  }

  // If it's already in date range format (YYYY-MM-DD:YYYY-MM-DD), pass through
  if (timeRange.includes(":")) {
    return timeRange;
  }

  const today = new Date();

  switch (timeRange) {
    case "last-2-years": {
      const twoYearsAgo = new Date(today);
      twoYearsAgo.setFullYear(today.getFullYear() - 2);
      return `${formatDateForAPI(twoYearsAgo)}:${formatDateForAPI(today)}`;
    }

    case "last-5-years": {
      const fiveYearsAgo = new Date(today);
      fiveYearsAgo.setFullYear(today.getFullYear() - 5);
      return `${formatDateForAPI(fiveYearsAgo)}:${formatDateForAPI(today)}`;
    }

    case "all-time": {
      // npm registry started in 2010
      const npmStart = new Date(2010, 0, 1);
      return `${formatDateForAPI(npmStart)}:${formatDateForAPI(today)}`;
    }

    default:
      return timeRange;
  }
};

const CACHE_TTL_SECONDS = 3600; // 1 hour

export const getPackageData = async (
  packageName: string,
  timeRange: string
): Promise<PackageData> => {
  const apiTimeRange = convertTimeRangeToAPIFormat(timeRange);
  const cacheKey = `package:${packageName}:${apiTimeRange}`;

  // Try to get from cache first (skip in development)
  if (!isDevelopment) {
    const cached = await redis.get<PackageData>(cacheKey);
    if (cached) {
      return cached;
    }
  }

  // Cache miss - fetch from npm API
  const response = await fetch(
    `https://api.npmjs.org/downloads/range/${apiTimeRange}/${packageName}`,
    {
      headers: {
        "Content-Type": "application/json",
      },
    }
  );

  if (!response.ok) {
    throw new Error("Failed to fetch package data");
  }

  const data: PackageData = await response.json();

  // Store in cache with TTL (async, after response, skip in development)
  if (!isDevelopment) {
    after(() => {
      redis.setex(cacheKey, CACHE_TTL_SECONDS, data);
    });
  }

  return data;
};
````

## File: actions/package/search.ts
````typescript
"use server";

import { Redis } from "@upstash/redis";
import { after } from "next/server";

const redis = Redis.fromEnv();
const isDevelopment = process.env.NODE_ENV === "development";

export interface NpmPackage {
  package: {
    name: string;
    version: string;
    description: string;
    links: {
      npm: string;
    };
  };
  score: {
    final: number;
  };
}

export interface NpmSearchResponse {
  objects: NpmPackage[];
  total: number;
}

const CACHE_TTL_SECONDS = 3600; // 1 hour

export const searchPackages = async (
  query: string
): Promise<NpmSearchResponse> => {
  const cacheKey = `search:${query.toLowerCase()}`;

  // Try to get from cache first (skip in development)
  if (!isDevelopment) {
    const cached = await redis.get<NpmSearchResponse>(cacheKey);
    if (cached) {
      return cached;
    }
  }

  // Cache miss - fetch from npm registry
  const response = await fetch(
    `https://registry.npmjs.com/-/v1/search?text=${encodeURIComponent(query)}&size=10`,
    {
      headers: {
        "Content-Type": "application/json",
      },
    }
  );

  if (!response.ok) {
    throw new Error("Failed to fetch packages");
  }

  const data: NpmSearchResponse = await response.json();

  // Store in cache with TTL (async, after response, skip in development)
  if (!isDevelopment) {
    after(() => {
      redis.setex(cacheKey, CACHE_TTL_SECONDS, data);
    });
  }

  return data;
};
````

## File: app/[package]/components/content.tsx
````typescript
"use client";

import useSWR from "swr";
import { getPackageData, type PackageData } from "@/actions/package/get";
import { ChartAreaInteractive } from "@/components/chart";
import { ChartLoading } from "@/components/chart-loading";
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import { useTimeRange } from "@/providers/filters";

interface ContentProps {
  packages: string[];
}

export const Content = ({ packages }: ContentProps) => {
  const [timeRange] = useTimeRange();

  const { data, error } = useSWR<PackageData[]>(
    [packages, timeRange],
    async ([pkgs, range]: [string[], string]) =>
      Promise.all(pkgs.map((pkg) => getPackageData(pkg, range)))
  );

  if (error) {
    return (
      <main className="flex size-full items-center justify-center overflow-hidden">
        <Alert
          className="max-w-md border-destructive bg-destructive/5"
          variant="destructive"
        >
          <AlertTitle>Error loading package data</AlertTitle>
          <AlertDescription>
            Could not find package &quot;{packages.join(", ")}&quot;.
          </AlertDescription>
        </Alert>
      </main>
    );
  }

  if (!data) {
    return (
      <main className="overflow-hidden">
        <ChartLoading />
      </main>
    );
  }

  return (
    <main className="overflow-hidden">
      <ChartAreaInteractive data={data} />
    </main>
  );
};
````

## File: app/[package]/components/footer.tsx
````typescript
"use client";

import { useRouter, useSearchParams } from "next/navigation";
import { PackageSearch } from "@/components/package-search";

interface PackageFooterProps {
  packages: string[];
}

export const PackageFooter = ({ packages }: PackageFooterProps) => {
  const router = useRouter();
  const searchParams = useSearchParams();

  const handlePackagesChange = (nextPackages: string[]) => {
    const pathname =
      nextPackages.length > 0
        ? `/${nextPackages.map(encodeURIComponent).join(",")}`
        : "/";
    const query = searchParams.toString();

    router.replace(query ? `${pathname}?${query}` : pathname, {
      scroll: false,
    });
  };

  return (
    <footer className="flex items-center justify-center p-4">
      <PackageSearch
        onPackagesChange={handlePackagesChange}
        packages={packages}
      />
    </footer>
  );
};
````

## File: app/[package]/components/header.tsx
````typescript
"use client";

import { SettingsIcon } from "lucide-react";
import useSWR from "swr";
import { getPackageData, type PackageData } from "@/actions/package/get";
import { GitHub } from "@/components/github";
import { GroupingSelector } from "@/components/grouping-selector";
import { Logo } from "@/components/logo";
import { MetricSelector } from "@/components/metric-selector";
import { Screenshot } from "@/components/screenshot";
import { ThemeToggle } from "@/components/theme-toggle";
import { TimeRangeSelector } from "@/components/time-range-selector";
import { Button } from "@/components/ui/button";
import { ButtonGroup } from "@/components/ui/button-group";
import {
  Drawer,
  DrawerClose,
  DrawerContent,
  DrawerDescription,
  DrawerFooter,
  DrawerHeader,
  DrawerTitle,
  DrawerTrigger,
} from "@/components/ui/drawer";
import { ZeroModeToggle } from "@/components/zero-mode-toggle";
import { useTimeRange } from "@/providers/filters";

interface PackageHeaderProps {
  packages: string[];
}

export const PackageHeader = ({ packages }: PackageHeaderProps) => {
  const [timeRange] = useTimeRange();

  const { data } = useSWR<PackageData[]>(
    [packages, timeRange],
    async ([pkgs, range]: [string[], string]) =>
      Promise.all(pkgs.map((pkg) => getPackageData(pkg, range)))
  );

  return (
    <header className="flex items-center justify-between">
      <Logo />
      <div className="flex items-center gap-2">
        <TimeRangeSelector className="hidden sm:flex" />
        <GroupingSelector className="hidden sm:flex" />
        {packages.length > 1 && <MetricSelector className="hidden sm:flex" />}
        <ZeroModeToggle className="hidden sm:inline-flex" />

        <ButtonGroup className="sm:hidden">
          <Drawer>
            <DrawerTrigger asChild>
              <Button className="shadow-none" size="icon" variant="outline">
                <SettingsIcon />
              </Button>
            </DrawerTrigger>
            <DrawerContent>
              <div className="mx-auto w-full max-w-sm">
                <DrawerHeader>
                  <DrawerTitle>Settings</DrawerTitle>
                  <DrawerDescription>Customize your chart.</DrawerDescription>
                </DrawerHeader>
                <div className="flex flex-col gap-2 p-4 pb-0">
                  <TimeRangeSelector className="w-full" />
                  <GroupingSelector className="w-full" />
                  <div className="flex items-center gap-2">
                    {packages.length > 1 && <MetricSelector />}
                    <ZeroModeToggle showLabel size="sm" />
                  </div>
                </div>
                <DrawerFooter>
                  <DrawerClose asChild>
                    <Button variant="outline">Close</Button>
                  </DrawerClose>
                </DrawerFooter>
              </div>
            </DrawerContent>
          </Drawer>
          <ThemeToggle />
        </ButtonGroup>

        <ButtonGroup className="hidden sm:flex">
          <Screenshot data={data} />
          <ThemeToggle />
        </ButtonGroup>

        <GitHub />
      </div>
    </header>
  );
};
````

## File: app/[package]/page.tsx
````typescript
import type { Metadata } from "next";
import { parseZeroMode } from "@/lib/download-data";
import { Content } from "./components/content";
import { PackageFooter } from "./components/footer";
import { PackageHeader } from "./components/header";

interface PackagePageProps {
  params: Promise<{
    package: string;
  }>;
  searchParams: Promise<{
    timeRange?: string;
    grouping?: string;
    metric?: string;
    zeroMode?: string;
  }>;
}

const protocol = process.env.NODE_ENV === "development" ? "http" : "https";
const baseUrl = new URL(
  `${protocol}://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`
);

export const generateMetadata = async ({
  params,
  searchParams,
}: PackagePageProps): Promise<Metadata> => {
  const { package: packageParam } = await params;
  const { timeRange, grouping, metric, zeroMode } = await searchParams;
  const packages = decodeURIComponent(packageParam).split(",");
  const title = packages.length > 1 ? packages.join(" vs ") : packages[0];

  const ogUrl = new URL("/og", baseUrl);
  ogUrl.searchParams.set("q", packages.join(","));
  ogUrl.searchParams.set("zeroMode", parseZeroMode(zeroMode));
  if (timeRange) {
    ogUrl.searchParams.set("timeRange", timeRange);
  }
  if (grouping) {
    ogUrl.searchParams.set("grouping", grouping);
  }
  if (metric) {
    ogUrl.searchParams.set("metric", metric);
  }

  return {
    title: `${title} - npm.bet`,
    description: `Download statistics for ${title} on npm`,
    openGraph: {
      title: `${title} - npm.bet`,
      description: `Download statistics for ${title} on npm`,
      images: [
        {
          url: ogUrl.toString(),
          width: 1200,
          height: 630,
        },
      ],
    },
  };
};

const PackagePage = async ({ params }: PackagePageProps) => {
  const { package: packageParam } = await params;
  const packages = decodeURIComponent(packageParam).split(",");

  return (
    <div className="grid h-dvh grid-rows-[2rem_1fr_2rem] gap-4 overflow-hidden p-4">
      <PackageHeader packages={packages} />
      <Content packages={packages} />
      <PackageFooter packages={packages} />
    </div>
  );
};

export default PackagePage;
````

## File: app/og/route.tsx
````typescript
import { ImageResponse } from "next/og";
import type { NextRequest } from "next/server";
import { getPackageData } from "@/actions/package/get";
import { shouldRemoveIncompleteDate } from "@/lib/chart-utils";
import {
  type PreparedDownload,
  parseZeroMode,
  prepareDownloadData,
} from "@/lib/download-data";

type StaticDownload = PreparedDownload & { denominatorEstimated?: boolean };

export const contentType = "image/png";

const fontRegex = /src: url\((.+)\) format\('(opentype|truetype)'\)/;

const loadGoogleFont = async (font: string, text: string, weights: string) => {
  const url = `https://fonts.googleapis.com/css2?family=${font}:wght@${weights}&text=${encodeURIComponent(text)}`;
  const css = await (await fetch(url)).text();
  const resource = css.match(fontRegex);

  if (resource) {
    const response = await fetch(resource[1]);
    if (response.status === 200) {
      return await response.arrayBuffer();
    }
  }

  throw new Error("failed to load font data");
};

// Colors for OG image (CSS variables don't work in OG images)
const colors = ["#f54a00", "#009689", "#104e64", "#ffba00", "#fd9a00"];

interface Point {
  x: number;
  y: number;
}

const createSvgPath = (points: Point[]): string => {
  if (points.length === 0) {
    return "";
  }

  const commands = points.map((point, index) => {
    const command = index === 0 ? "M" : "L";
    return `${command} ${point.x} ${point.y}`;
  });

  return commands.join(" ");
};

const normalizeData = (
  data: StaticDownload[],
  width: number,
  height: number,
  maxDownloads: number
) => {
  if (data.length === 0) {
    return [];
  }

  const padding = 40;
  const chartWidth = width - padding * 2;
  const chartHeight = height - padding * 2;

  return data.map((item, index) => ({
    ...item,
    x:
      padding +
      (data.length === 1 ? 0.5 : index / (data.length - 1)) * chartWidth,
    y:
      padding +
      chartHeight -
      (item.downloads / (maxDownloads || 1)) * chartHeight,
  }));
};

const formatNumber = (num: number): string => {
  if (num >= 1_000_000) {
    return `${(num / 1_000_000).toFixed(1)}M`;
  }
  if (num >= 1000) {
    return `${(num / 1000).toFixed(0)}K`;
  }
  return num.toString();
};

const removeCurrentPeriodFromData = <T extends { date: string }>(
  data: T[],
  grouping: string
): T[] => {
  if (data.length <= 1) {
    return data;
  }

  const lastDateString = data.at(-1)?.date;
  if (!lastDateString) {
    return data;
  }

  if (shouldRemoveIncompleteDate(lastDateString, grouping)) {
    return data.slice(0, -1);
  }

  return data;
};

const computeShareData = (
  groupedPackageData: {
    package: string;
    downloads: StaticDownload[];
  }[]
) => {
  let grandTotalDownloads = 0;
  const packageTotals: Record<string, number> = {};
  const downloadsByDate: Record<string, Record<string, number>> = {};
  const overallShareByPackage: Record<string, number> = {};
  const estimatedDates = new Set<string>();

  for (const pkg of groupedPackageData) {
    const pkgTotal = pkg.downloads.reduce((s, d) => s + d.downloads, 0);
    packageTotals[pkg.package] = pkgTotal;
    grandTotalDownloads += pkgTotal;
    for (const d of pkg.downloads) {
      if (d.estimatedDays > 0) {
        estimatedDates.add(d.date);
      }
      if (!downloadsByDate[d.date]) {
        downloadsByDate[d.date] = {};
      }
      downloadsByDate[d.date][pkg.package] = d.downloads;
    }
  }

  for (const pkg of groupedPackageData) {
    overallShareByPackage[pkg.package] =
      grandTotalDownloads > 0
        ? Number(
            ((packageTotals[pkg.package] / grandTotalDownloads) * 100).toFixed(
              1
            )
          )
        : 0;
    pkg.downloads = pkg.downloads.map((d) => {
      const dateData = downloadsByDate[d.date] ?? {};
      const total = Object.values(dateData).reduce((s, v) => s + v, 0);
      return {
        ...d,
        denominatorEstimated: estimatedDates.has(d.date),
        downloads:
          total > 0 ? Number(((d.downloads / total) * 100).toFixed(1)) : 0,
      };
    });
  }

  return overallShareByPackage;
};

export const GET = async (request: NextRequest) => {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get("q");

  const config = {
    width: 1200,
    height: 628,
    fonts: [
      {
        name: "Geist",
        data: await loadGoogleFont(
          "Geist",
          "abcdefghijklmnopqrstuvwxyz.0123456789",
          "400"
        ),
        weight: 400,
      },
      {
        name: "Geist",
        data: await loadGoogleFont(
          "Geist",
          "abcdefghijklmnopqrstuvwxyz.0123456789",
          "600"
        ),
        weight: 600,
      },
    ],
  };

  if (!query) {
    return new ImageResponse(
      <div tw="bg-[#f5f5f5] w-full h-full flex items-center justify-center">
        <div
          style={{ gap: "16px" }}
          tw="flex items-center justify-center shrink-0 mt-2"
        >
          {/** biome-ignore lint/a11y/noSvgWithoutTitle: "npm.bet logo" */}
          <svg
            fill="none"
            height={48}
            stroke="currentColor"
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            viewBox="0 0 24 24"
            width={48}
            xmlns="http://www.w3.org/2000/svg"
          >
            <path d="M12 22v-9" />
            <path d="M15.17 2.21a1.67 1.67 0 0 1 1.63 0L21 4.57a1.93 1.93 0 0 1 0 3.36L8.82 14.79a1.655 1.655 0 0 1-1.64 0L3 12.43a1.93 1.93 0 0 1 0-3.36z" />
            <path d="M20 13v3.87a2.06 2.06 0 0 1-1.11 1.83l-6 3.08a1.93 1.93 0 0 1-1.78 0l-6-3.08A2.06 2.06 0 0 1 4 16.87V13" />
            <path d="M21 12.43a1.93 1.93 0 0 0 0-3.36L8.83 2.2a1.64 1.64 0 0 0-1.63 0L3 4.57a1.93 1.93 0 0 0 0 3.36l12.18 6.86a1.636 1.636 0 0 0 1.63 0z" />
          </svg>
          <p tw="font-semibold text-5xl">npm.bet</p>
        </div>
      </div>,
      config as never
    );
  }

  const packages = query.split(",").slice(0, 5); // Limit to 5 packages
  const timeRange = searchParams.get("timeRange") ?? "last-year";
  const grouping = searchParams.get("grouping") ?? "week";
  const metric = searchParams.get("metric") ?? "downloads";
  const isShare = metric === "share";
  const zeroMode = parseZeroMode(searchParams.get("zeroMode"));
  const removeCurrentPeriod =
    searchParams.get("removeCurrentPeriod") !== "false";

  try {
    // Fetch data for all packages
    const packageDataArray = await Promise.all(
      packages.map((pkg) => getPackageData(pkg.trim(), timeRange))
    );

    // Apply grouping to each package's data
    const groupedPackageData = packageDataArray.map((pkg) => {
      const grouped = prepareDownloadData(pkg.downloads, grouping, zeroMode);
      const processed = removeCurrentPeriod
        ? removeCurrentPeriodFromData(grouped, grouping)
        : grouped;
      return {
        ...pkg,
        downloads: processed,
      };
    });

    const hasEstimates = groupedPackageData.some((pkg) =>
      pkg.downloads.some((item) => item.estimatedDays > 0)
    );
    const estimateDisclosure = isShare
      ? "Includes estimated downloads; share denominators include estimates"
      : "Includes estimated downloads";
    const chartTitle = isShare
      ? "npm package download share (%)"
      : "npm package download comparison chart";

    // Compute overall share stats and convert to share (%) if metric is "share"
    const overallShareByPackage = isShare
      ? computeShareData(groupedPackageData)
      : {};

    // Find the maximum value across all packages after grouping
    const maxDownloads = isShare
      ? Math.min(
          100,
          Math.ceil(
            (Math.max(
              0,
              ...groupedPackageData.flatMap((pkg) =>
                pkg.downloads.map((d) => d.downloads)
              )
            ) +
              5) /
              10
          ) * 10
        )
      : Math.max(
          0,
          ...groupedPackageData.flatMap((pkg) =>
            pkg.downloads.map((d) => d.downloads)
          )
        );

    // Create SVG chart dimensions
    const chartWidth = 1100;
    const chartHeight = hasEstimates ? 416 : 440;

    const chart = (
      <svg
        aria-label={`${chartTitle}${hasEstimates ? `. ${estimateDisclosure}` : ""}`}
        height={chartHeight}
        role="img"
        viewBox={`0 0 ${chartWidth} ${chartHeight}`}
        width={chartWidth}
      >
        <title>{chartTitle}</title>
        <desc>
          {hasEstimates
            ? `${estimateDisclosure}. Hollow dashed circles mark estimate-affected points.`
            : "Reported downloads only."}
        </desc>
        {/* Grid lines */}
        <line
          stroke="#e5e7eb"
          strokeWidth="1"
          x1="40"
          x2={chartWidth - 40}
          y1={chartHeight - 40}
          y2={chartHeight - 40}
        />
        {[0.25, 0.5, 0.75].map((ratio) => (
          <line
            key={ratio}
            opacity="0.5"
            stroke="#e5e7eb"
            strokeDasharray="4 4"
            strokeWidth="1"
            x1="40"
            x2={chartWidth - 40}
            y1={40 + (chartHeight - 80) * (1 - ratio)}
            y2={40 + (chartHeight - 80) * (1 - ratio)}
          />
        ))}

        {/* Draw lines for each package */}
        {groupedPackageData.map((pkg, index) => {
          const points = normalizeData(
            pkg.downloads,
            chartWidth,
            chartHeight,
            maxDownloads
          );
          const path = createSvgPath(points);

          return (
            <g key={pkg.package}>
              <path
                d={path}
                fill="none"
                stroke={colors[index % colors.length]}
                strokeWidth="3"
              />
              {points
                .filter(
                  (point) =>
                    point.estimatedDays > 0 || point.denominatorEstimated
                )
                .map((point) => (
                  <circle
                    cx={point.x}
                    cy={point.y}
                    fill="#ffffff"
                    key={point.date}
                    r="5"
                    stroke={colors[index % colors.length]}
                    strokeDasharray="3 2"
                    strokeWidth="2"
                  >
                    <title>
                      {`${pkg.package}, ${point.date}: ${point.downloads}${isShare ? "% share" : " downloads"}; ${point.reportedDownloads} reported downloads; ${point.estimatedDays} estimated days${point.denominatorEstimated ? "; share denominator includes estimates" : ""}`}
                    </title>
                  </circle>
                ))}
            </g>
          );
        })}
      </svg>
    );

    const legend = (
      <div
        style={{ gap: "24px" }}
        tw="flex items-center justify-center flex-wrap mb-5"
      >
        {groupedPackageData.map((pkg, index) => {
          const totalDownloads = pkg.downloads.reduce(
            (sum, d) => sum + d.downloads,
            0
          );
          const displayValue = isShare
            ? `${overallShareByPackage[pkg.package] ?? 0}%`
            : formatNumber(totalDownloads);
          const estimated = isShare
            ? hasEstimates
            : pkg.downloads.some((point) => point.estimatedDays > 0);
          return (
            <div
              key={pkg.package}
              style={{ gap: "8px" }}
              tw="flex items-center"
            >
              <div
                style={{
                  background: colors[index % colors.length],
                }}
                tw="w-4 h-4 rounded-full"
              />
              <span tw="text-lg font-semibold text-[#1f2937]">
                {pkg.package}
              </span>
              <span tw="text-lg font-semibold text-[#737373]">
                {estimated ? `≈ ${displayValue}` : displayValue}
              </span>
            </div>
          );
        })}
      </div>
    );

    const logo = (
      <div
        style={{ gap: "8px" }}
        tw="flex items-center justify-center shrink-0 mt-2"
      >
        {/** biome-ignore lint/a11y/noSvgWithoutTitle: "npm.bet logo" */}
        <svg
          fill="none"
          height={16}
          stroke="currentColor"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          viewBox="0 0 24 24"
          width={16}
          xmlns="http://www.w3.org/2000/svg"
        >
          <path d="M12 22v-9" />
          <path d="M15.17 2.21a1.67 1.67 0 0 1 1.63 0L21 4.57a1.93 1.93 0 0 1 0 3.36L8.82 14.79a1.655 1.655 0 0 1-1.64 0L3 12.43a1.93 1.93 0 0 1 0-3.36z" />
          <path d="M20 13v3.87a2.06 2.06 0 0 1-1.11 1.83l-6 3.08a1.93 1.93 0 0 1-1.78 0l-6-3.08A2.06 2.06 0 0 1 4 16.87V13" />
          <path d="M21 12.43a1.93 1.93 0 0 0 0-3.36L8.83 2.2a1.64 1.64 0 0 0-1.63 0L3 4.57a1.93 1.93 0 0 0 0 3.36l12.18 6.86a1.636 1.636 0 0 0 1.63 0z" />
        </svg>
        <p tw="font-semibold text-lg">npm.bet</p>
      </div>
    );

    return new ImageResponse(
      <div tw="bg-[#f5f5f5] w-full h-full flex flex-col p-12 overflow-hidden">
        <div tw="flex flex-col bg-white rounded-lg border border-[#e5e5e5] overflow-hidden">
          {chart}
          {legend}
          {hasEstimates ? (
            <div
              style={{ gap: "8px", marginBottom: "8px", fontSize: "14px" }}
              tw="flex items-center justify-center text-[#737373]"
            >
              <svg
                aria-label="Estimate-affected point"
                height={16}
                role="img"
                viewBox="0 0 16 16"
                width={16}
              >
                <title>Estimate-affected point</title>
                <circle
                  cx="8"
                  cy="8"
                  fill="#ffffff"
                  r="5"
                  stroke="#737373"
                  strokeDasharray="3 2"
                  strokeWidth="2"
                />
              </svg>
              <span>{estimateDisclosure}</span>
            </div>
          ) : null}
        </div>
        {logo}
      </div>,
      config as never
    );
  } catch (error) {
    console.error(error);

    // Fallback error image
    return new ImageResponse(
      <div
        style={{
          fontSize: 48,
          background: "white",
          width: "100%",
          height: "100%",
          display: "flex",
          flexDirection: "column",
          alignItems: "center",
          justifyContent: "center",
          color: "#1f2937",
        }}
      >
        <div style={{ marginBottom: "20px", fontSize: "64px" }}>📦</div>
        <div style={{ fontWeight: 600 }}>{packages.join(" vs ")}</div>
        <div
          style={{
            fontSize: "24px",
            color: "#6b7280",
            marginTop: "10px",
          }}
        >
          npm.bet
        </div>
      </div>,
      config as never
    );
  }
};
````

## File: app/svg/route.ts
````typescript
import { type NextRequest, NextResponse } from "next/server";
import { getPackageData, type PackageData } from "@/actions/package/get";
import {
  escapeXml,
  formatNumber,
  shouldRemoveIncompleteDate,
} from "@/lib/chart-utils";
import {
  type PreparedDownload,
  parseZeroMode,
  prepareDownloadData,
  type ZeroMode,
} from "@/lib/download-data";

const colors = [
  "#3b82f6", // blue
  "#10b981", // green
  "#f59e0b", // amber
  "#ef4444", // red
  "#8b5cf6", // purple
];

interface ChartDataPoint {
  date: string;
  packages: Record<
    string,
    PreparedDownload & { denominatorEstimated?: boolean }
  >;
}

const mergePackageData = (
  data: PackageData[],
  grouping: string,
  zeroMode: ZeroMode,
  isShare: boolean
): ChartDataPoint[] => {
  const allDates = new Set<string>();
  const packagesByDate: Record<string, Record<string, PreparedDownload>> = {};

  for (const pkg of data) {
    const grouped = prepareDownloadData(pkg.downloads, grouping, zeroMode);
    for (const item of grouped) {
      allDates.add(item.date);
      if (!packagesByDate[item.date]) {
        packagesByDate[item.date] = {};
      }
      packagesByDate[item.date][pkg.package] = item;
    }
  }

  const sortedDates = Array.from(allDates).sort();

  // Remove the most recent incomplete data point
  if (sortedDates.length > 1) {
    const lastDateString = sortedDates.at(-1);
    if (
      lastDateString &&
      shouldRemoveIncompleteDate(lastDateString, grouping)
    ) {
      sortedDates.pop();
    }
  }

  return sortedDates.map((date) => {
    const packages = packagesByDate[date];
    if (!isShare) {
      return { date, packages };
    }
    const total = Object.values(packages).reduce(
      (sum, item) => sum + item.downloads,
      0
    );
    const denominatorEstimated = Object.values(packages).some(
      (item) => item.estimatedDays > 0
    );
    return {
      date,
      packages: Object.fromEntries(
        Object.entries(packages).map(([name, item]) => [
          name,
          {
            ...item,
            downloads: total > 0 ? (item.downloads / total) * 100 : 0,
            denominatorEstimated,
          },
        ])
      ),
    };
  });
};

const generateSVGChart = (
  packageData: PackageData[],
  grouping: string,
  zeroMode: ZeroMode,
  isShare: boolean
): string => {
  const chartData = mergePackageData(packageData, grouping, zeroMode, isShare);
  const hasEstimates = chartData.some((row) =>
    Object.values(row.packages).some((item) => item.estimatedDays > 0)
  );
  const estimateDisclosure = isShare
    ? "Includes estimated downloads; share denominators include estimates"
    : "Includes estimated downloads";
  const chartTitle = isShare
    ? "npm package download share (%)"
    : "npm package downloads";
  const width = 800;
  // Adjust height based on number of packages for legend
  const legendItemHeight = 18;
  const legendHeight =
    Math.max(packageData.length * legendItemHeight, 20) +
    (hasEstimates ? legendItemHeight : 0);
  const baseHeight = 400;
  const height = baseHeight + legendHeight;

  const padding = {
    top: 40,
    right: 40,
    bottom: 60 + legendHeight,
    left: 60,
  };
  const chartWidth = width - padding.left - padding.right;
  const chartHeight = baseHeight - 100; // Fixed chart height (400 - top 40 - bottom 60)

  if (chartData.length === 0) {
    return `
      <svg xmlns="http://www.w3.org/2000/svg" width="${width}" height="${height}">
        <rect width="${width}" height="${height}" fill="#ffffff"/>
        <text x="${width / 2}" y="${height / 2}" text-anchor="middle" font-family="Arial" font-size="16" fill="#666">
          No data available
        </text>
      </svg>
    `;
  }

  // Calculate scales
  const maxDownloads = isShare
    ? 100
    : Math.max(
        0,
        ...chartData.flatMap((d) =>
          Object.values(d.packages).map((item) => item.downloads)
        )
      );

  const xScale = (index: number) =>
    padding.left +
    (chartData.length === 1 ? 0.5 : index / (chartData.length - 1)) *
      chartWidth;

  const yScale = (value: number) =>
    padding.top + chartHeight - (value / (maxDownloads || 1)) * chartHeight;

  // Generate Y-axis ticks
  const yTicks = 5;
  const yTickValues = Array.from({ length: yTicks }, (_, i) =>
    Math.round((maxDownloads / (yTicks - 1)) * i)
  );

  // Helper function to create smooth curves using Cardinal spline
  const createSmoothPath = (points: { x: number; y: number }[]): string => {
    if (points.length === 0) {
      return "";
    }
    if (points.length === 1) {
      return `M ${points[0].x},${points[0].y}`;
    }
    if (points.length === 2) {
      return `M ${points[0].x},${points[0].y} L ${points[1].x},${points[1].y}`;
    }

    let path = `M ${points[0].x},${points[0].y}`;

    // Cardinal spline tension (0 = Catmull-Rom, 0.5 = smoother)
    const tension = 0.5;

    for (let i = 0; i < points.length - 1; i++) {
      const p0 = points[Math.max(0, i - 1)];
      const p1 = points[i];
      const p2 = points[i + 1];
      const p3 = points[Math.min(points.length - 1, i + 2)];

      // Calculate control points for cubic Bezier curve
      const cp1x = p1.x + ((p2.x - p0.x) / 6) * tension;
      const cp1y = p1.y + ((p2.y - p0.y) / 6) * tension;
      const cp2x = p2.x - ((p3.x - p1.x) / 6) * tension;
      const cp2y = p2.y - ((p3.y - p1.y) / 6) * tension;

      path += ` C ${cp1x},${cp1y} ${cp2x},${cp2y} ${p2.x},${p2.y}`;
    }

    return path;
  };

  // Generate smooth lines for each package
  const lines = packageData.map((pkg, pkgIndex) => {
    const points = chartData
      .map((d, i) => {
        const item = d.packages[pkg.package];
        if (!item) {
          return null;
        }
        return { x: xScale(i), y: yScale(item.downloads) };
      })
      .filter((p): p is { x: number; y: number } => p !== null);

    const pathData = createSmoothPath(points);
    const markers = chartData
      .map((row, index) => {
        const item = row.packages[pkg.package];
        if (!(item && (item.estimatedDays > 0 || item.denominatorEstimated))) {
          return "";
        }
        const label = `${pkg.package}, ${row.date}: ${isShare ? `${item.downloads.toFixed(1)}% share` : `${item.downloads} downloads`}; ${item.reportedDownloads} reported downloads; ${item.estimatedDays} estimated days${item.denominatorEstimated ? "; share denominator includes estimates" : ""}`;
        return `<circle cx="${xScale(index)}" cy="${yScale(item.downloads)}" r="4" fill="#ffffff" stroke="${colors[pkgIndex % colors.length]}" stroke-width="2" stroke-dasharray="2 2"><title>${escapeXml(label)}</title></circle>`;
      })
      .join("");

    return `
      <path
        fill="none"
        stroke="${colors[pkgIndex % colors.length]}"
        stroke-width="2"
        d="${pathData}"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
      ${markers}
    `;
  });

  // Generate X-axis labels (show every nth label to avoid crowding)
  const xLabelStep = Math.ceil(chartData.length / 8);
  const xLabels = chartData
    .map((d, i) => {
      if (i % xLabelStep !== 0 && i !== chartData.length - 1) {
        return "";
      }
      const date = new Date(d.date);
      const label = date.toLocaleDateString("en-US", {
        month: "short",
        day: "numeric",
      });
      return `
        <text
          x="${xScale(i)}"
          y="${height - padding.bottom + 20}"
          text-anchor="middle"
          font-family="Arial"
          font-size="12"
          fill="#666"
        >
          ${label}
        </text>
      `;
    })
    .join("");

  // Generate Y-axis labels
  const yLabels = yTickValues
    .map(
      (value) => `
        <text
          x="${padding.left - 10}"
          y="${yScale(value)}"
          text-anchor="end"
          font-family="Arial"
          font-size="12"
          fill="#666"
          alignment-baseline="middle"
        >
          ${isShare ? `${value}%` : formatNumber(value)}
        </text>
        <line
          x1="${padding.left}"
          y1="${yScale(value)}"
          x2="${width - padding.right}"
          y2="${yScale(value)}"
          stroke="#e5e5e5"
          stroke-width="1"
        />
      `
    )
    .join("");

  // Generate legend (vertical layout at the bottom, outside chart area)
  const legendStartY = height - padding.bottom + 48; // Start below x-axis labels
  const legend = packageData
    .map(
      (pkg, i) => `
        <g transform="translate(${padding.left}, ${legendStartY + i * legendItemHeight})">
          <line x1="0" y1="0" x2="20" y2="0" stroke="${colors[i % colors.length]}" stroke-width="2"/>
          <text x="25" y="0" font-family="Arial" font-size="12" fill="#333" alignment-baseline="middle">
            ${escapeXml(pkg.package)}
          </text>
        </g>
      `
    )
    .join("");

  const estimateLegend = hasEstimates
    ? `<g transform="translate(${padding.left}, ${legendStartY + packageData.length * legendItemHeight})">
        <circle cx="10" cy="0" r="4" fill="#ffffff" stroke="#666" stroke-width="2" stroke-dasharray="2 2"/>
        <text x="25" y="0" font-family="Arial" font-size="12" fill="#333" alignment-baseline="middle">${estimateDisclosure}</text>
      </g>`
    : "";

  return `
    <svg xmlns="http://www.w3.org/2000/svg" width="${width}" height="${height}" role="img" aria-labelledby="chart-title chart-description">
      <title id="chart-title">${chartTitle}</title>
      <desc id="chart-description">${escapeXml(packageData.map((pkg) => pkg.package).join(" vs "))}. ${hasEstimates ? `${estimateDisclosure}. Hollow dashed circles mark estimate-affected points.` : "Reported downloads only."}</desc>
      <rect width="${width}" height="${height}" fill="#ffffff"/>

      <!-- Brand Label with Logo -->
      <g transform="translate(${width / 2 - 32}, 12)">
        <g transform="scale(0.18) translate(-40, 4)">
          <svg viewBox="0 0 76 65" width="76" height="65">
            <path d="M37.5274 0L75.0548 65H0L37.5274 0Z" fill="#000"/>
          </svg>
        </g>
        <text
          x="12"
          y="8"
          font-family="Arial"
          font-size="12"
          fill="#737373"
          opacity="0.5"
          alignment-baseline="middle"
        >
          /
        </text>
        <text
          x="20"
          y="8"
          font-family="Arial"
          font-size="12"
          fill="#000"
          alignment-baseline="middle"
        >
          npm.bet
        </text>
      </g>

      <!-- Y-axis grid and labels -->
      ${yLabels}

      <!-- Chart lines -->
      ${lines.join("")}

      <!-- X-axis labels -->
      ${xLabels}

      <!-- Axes -->
      <line
        x1="${padding.left}"
        y1="${padding.top}"
        x2="${padding.left}"
        y2="${height - padding.bottom}"
        stroke="#333"
        stroke-width="1"
      />
      <line
        x1="${padding.left}"
        y1="${height - padding.bottom}"
        x2="${width - padding.right}"
        y2="${height - padding.bottom}"
        stroke="#333"
        stroke-width="1"
      />

      <!-- Legend -->
      ${legend}
      ${estimateLegend}
    </svg>
  `;
};

export async function GET(request: NextRequest) {
  try {
    const searchParams = request.nextUrl.searchParams;
    const packagesParam = searchParams.get("q");
    const timeRange = searchParams.get("timeRange") || "last-year";
    const grouping = searchParams.get("grouping") || "week";
    const zeroMode = parseZeroMode(searchParams.get("zeroMode"));
    const isShare = searchParams.get("metric") === "share";

    if (!packagesParam) {
      return new NextResponse(
        `
        <svg xmlns="http://www.w3.org/2000/svg" width="800" height="400">
          <rect width="800" height="400" fill="#ffffff"/>
          <text x="400" y="200" text-anchor="middle" font-family="Arial" font-size="16" fill="#666">
            Please provide package name(s) via ?q=package-name
          </text>
        </svg>
      `,
        {
          status: 400,
          headers: {
            "Content-Type": "image/svg+xml",
            "Cache-Control": "public, max-age=3600", // Cache for 1 hour
          },
        }
      );
    }

    // Parse packages (support comma-separated list, max 5)
    const packages = packagesParam.split(",").filter(Boolean).slice(0, 5);

    // Fetch data for all packages
    const packageData = await Promise.all(
      packages.map((pkg) => getPackageData(pkg.trim(), timeRange))
    );

    // Generate SVG
    const svg = generateSVGChart(packageData, grouping, zeroMode, isShare);

    return new NextResponse(svg, {
      headers: {
        "Content-Type": "image/svg+xml",
        "Cache-Control": "public, max-age=3600", // Cache for 1 hour
      },
    });
  } catch (error) {
    console.error("Error generating SVG chart:", error);

    return new NextResponse(
      `
      <svg xmlns="http://www.w3.org/2000/svg" width="800" height="400">
        <rect width="800" height="400" fill="#ffffff"/>
        <text x="400" y="200" text-anchor="middle" font-family="Arial" font-size="16" fill="#ff0000">
          Error generating chart
        </text>
      </svg>
    `,
      {
        status: 500,
        headers: {
          "Content-Type": "image/svg+xml",
        },
      }
    );
  }
}
````

## File: app/globals.css
````css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-sidebar-ring: var(--sidebar-ring);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar: var(--sidebar);
  --color-chart-5: var(--chart-5);
  --color-chart-4: var(--chart-4);
  --color-chart-3: var(--chart-3);
  --color-chart-2: var(--chart-2);
  --color-chart-1: var(--chart-1);
  --color-ring: var(--ring);
  --color-input: var(--input);
  --color-border: var(--border);
  --color-destructive: var(--destructive);
  --color-accent-foreground: var(--accent-foreground);
  --color-accent: var(--accent);
  --color-muted-foreground: var(--muted-foreground);
  --color-muted: var(--muted);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-secondary: var(--secondary);
  --color-primary-foreground: var(--primary-foreground);
  --color-primary: var(--primary);
  --color-popover-foreground: var(--popover-foreground);
  --color-popover: var(--popover);
  --color-card-foreground: var(--card-foreground);
  --color-card: var(--card);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}

:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --chart-1: oklch(0.646 0.222 41.116);
  --chart-2: oklch(0.6 0.118 184.704);
  --chart-3: oklch(0.398 0.07 227.392);
  --chart-4: oklch(0.828 0.189 84.429);
  --chart-5: oklch(0.769 0.188 70.08);
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.145 0 0);
  --sidebar-primary: oklch(0.205 0 0);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.97 0 0);
  --sidebar-accent-foreground: oklch(0.205 0 0);
  --sidebar-border: oklch(0.922 0 0);
  --sidebar-ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.205 0 0);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.556 0 0);
  --chart-1: oklch(0.488 0.243 264.376);
  --chart-2: oklch(0.696 0.17 162.48);
  --chart-3: oklch(0.769 0.188 70.08);
  --chart-4: oklch(0.627 0.265 303.9);
  --chart-5: oklch(0.645 0.246 16.439);
  --sidebar: oklch(0.205 0 0);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.269 0 0);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.556 0 0);
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-sidebar text-foreground;
  }
}
````

## File: app/layout.tsx
````typescript
import type { Metadata, Viewport } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import "./globals.css";
import { Analytics } from "@vercel/analytics/next";
import { ThemeProvider } from "next-themes";
import { NuqsAdapter } from "nuqs/adapters/next/app";
import { type ReactNode, Suspense } from "react";
import { cn } from "@/lib/utils";

const geistSans = Geist({
  variable: "--font-sans",
  subsets: ["latin"],
});

const geistMono = Geist_Mono({
  variable: "--font-mono",
  subsets: ["latin"],
});

const title = "npm.bet";
const description = "visualize and compare your npm package downloads.";
const authors = [{ name: "Vercel", url: "https://vercel.com" }];
const creator = "Vercel";
const publisher = "Vercel";
const twitterHandle = "@vercel";

export const metadata: Metadata = {
  title,
  description,
  authors,
  creator,
  formatDetection: {
    telephone: false,
  },
  appleWebApp: {
    capable: true,
    statusBarStyle: "default",
    title,
  },
  openGraph: {
    title,
    description,
    type: "website",
    siteName: title,
    locale: "en_US",
  },
  publisher,
  twitter: {
    card: "summary_large_image",
    creator: twitterHandle,
  },
};

export const viewport: Viewport = {
  width: "device-width",
  initialScale: 1,
  maximumScale: 1,
  userScalable: false,
};

interface RootLayoutProps {
  children: ReactNode;
}

const RootLayout = ({ children }: RootLayoutProps) => (
  <html lang="en" suppressHydrationWarning>
    <body className={cn(geistSans.variable, geistMono.variable, "antialiased")}>
      <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
        <NuqsAdapter>
          <Suspense>{children}</Suspense>
        </NuqsAdapter>
        <Analytics />
      </ThemeProvider>
    </body>
  </html>
);

export default RootLayout;
````

## File: app/page.tsx
````typescript
import type { Metadata } from "next";
import { Footer } from "@/components/footer";
import { Header } from "@/components/header";
import { Main } from "@/components/main";
import { parseZeroMode } from "@/lib/download-data";

interface HomeProps {
  searchParams: Promise<{
    q: string;
    timeRange?: string;
    grouping?: string;
    metric?: string;
    zeroMode?: string;
  }>;
}

const protocol = process.env.NODE_ENV === "development" ? "http" : "https";
const baseUrl = new URL(
  `${protocol}://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`
);

export const generateMetadata = async ({ searchParams }: HomeProps) => {
  const { q, timeRange, grouping, metric, zeroMode } = await searchParams;

  if (!q) {
    return {
      title: "npm.bet",
    };
  }

  const packages = q.split(",");

  // Build OG image URL with query params
  const ogUrl = new URL("/og", baseUrl);
  ogUrl.searchParams.set("q", q);
  ogUrl.searchParams.set("zeroMode", parseZeroMode(zeroMode));
  if (timeRange) {
    ogUrl.searchParams.set("timeRange", timeRange);
  }
  if (grouping) {
    ogUrl.searchParams.set("grouping", grouping);
  }
  if (metric) {
    ogUrl.searchParams.set("metric", metric);
  }

  const metadata: Metadata = {
    title: `${packages.join(" vs ")} - npm.bet`,
    openGraph: {
      images: [
        {
          url: ogUrl.toString(),
          width: 1200,
          height: 630,
        },
      ],
    },
  };

  return metadata;
};

const Home = async () => (
  <div className="grid h-dvh grid-rows-[2rem_1fr_2rem] gap-4 overflow-hidden p-4">
    <Header />
    <Main />
    <Footer />
  </div>
);

export default Home;
````

## File: components/ui/accordion.tsx
````typescript
"use client"

import * as React from "react"
import { Accordion as AccordionPrimitive } from "radix-ui"
import { ChevronDownIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Accordion({
  ...props
}: React.ComponentProps<typeof AccordionPrimitive.Root>) {
  return <AccordionPrimitive.Root data-slot="accordion" {...props} />
}

function AccordionItem({
  className,
  ...props
}: React.ComponentProps<typeof AccordionPrimitive.Item>) {
  return (
    <AccordionPrimitive.Item
      data-slot="accordion-item"
      className={cn("border-b last:border-b-0", className)}
      {...props}
    />
  )
}

function AccordionTrigger({
  className,
  children,
  ...props
}: React.ComponentProps<typeof AccordionPrimitive.Trigger>) {
  return (
    <AccordionPrimitive.Header className="flex">
      <AccordionPrimitive.Trigger
        data-slot="accordion-trigger"
        className={cn(
          "focus-visible:border-ring focus-visible:ring-ring/50 flex flex-1 items-start justify-between gap-4 rounded-md py-4 text-left text-sm font-medium transition-all outline-none hover:underline focus-visible:ring-[3px] disabled:pointer-events-none disabled:opacity-50 [&[data-state=open]>svg]:rotate-180",
          className
        )}
        {...props}
      >
        {children}
        <ChevronDownIcon className="text-muted-foreground pointer-events-none size-4 shrink-0 translate-y-0.5 transition-transform duration-200" />
      </AccordionPrimitive.Trigger>
    </AccordionPrimitive.Header>
  )
}

function AccordionContent({
  className,
  children,
  ...props
}: React.ComponentProps<typeof AccordionPrimitive.Content>) {
  return (
    <AccordionPrimitive.Content
      data-slot="accordion-content"
      className="data-[state=closed]:animate-accordion-up data-[state=open]:animate-accordion-down overflow-hidden text-sm"
      {...props}
    >
      <div className={cn("pt-0 pb-4", className)}>{children}</div>
    </AccordionPrimitive.Content>
  )
}

export { Accordion, AccordionItem, AccordionTrigger, AccordionContent }
````

## File: components/ui/alert-dialog.tsx
````typescript
"use client"

import * as React from "react"
import { AlertDialog as AlertDialogPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"
import { buttonVariants } from "@/components/ui/button"

function AlertDialog({
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Root>) {
  return <AlertDialogPrimitive.Root data-slot="alert-dialog" {...props} />
}

function AlertDialogTrigger({
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Trigger>) {
  return (
    <AlertDialogPrimitive.Trigger data-slot="alert-dialog-trigger" {...props} />
  )
}

function AlertDialogPortal({
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Portal>) {
  return (
    <AlertDialogPrimitive.Portal data-slot="alert-dialog-portal" {...props} />
  )
}

function AlertDialogOverlay({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Overlay>) {
  return (
    <AlertDialogPrimitive.Overlay
      data-slot="alert-dialog-overlay"
      className={cn(
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50",
        className
      )}
      {...props}
    />
  )
}

function AlertDialogContent({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Content>) {
  return (
    <AlertDialogPortal>
      <AlertDialogOverlay />
      <AlertDialogPrimitive.Content
        data-slot="alert-dialog-content"
        className={cn(
          "bg-background data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 fixed top-[50%] left-[50%] z-50 grid w-full max-w-[calc(100%-2rem)] translate-x-[-50%] translate-y-[-50%] gap-4 rounded-lg border p-6 shadow-lg duration-200 sm:max-w-lg",
          className
        )}
        {...props}
      />
    </AlertDialogPortal>
  )
}

function AlertDialogHeader({
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="alert-dialog-header"
      className={cn("flex flex-col gap-2 text-center sm:text-left", className)}
      {...props}
    />
  )
}

function AlertDialogFooter({
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="alert-dialog-footer"
      className={cn(
        "flex flex-col-reverse gap-2 sm:flex-row sm:justify-end",
        className
      )}
      {...props}
    />
  )
}

function AlertDialogTitle({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Title>) {
  return (
    <AlertDialogPrimitive.Title
      data-slot="alert-dialog-title"
      className={cn("text-lg font-semibold", className)}
      {...props}
    />
  )
}

function AlertDialogDescription({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Description>) {
  return (
    <AlertDialogPrimitive.Description
      data-slot="alert-dialog-description"
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

function AlertDialogAction({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Action>) {
  return (
    <AlertDialogPrimitive.Action
      className={cn(buttonVariants(), className)}
      {...props}
    />
  )
}

function AlertDialogCancel({
  className,
  ...props
}: React.ComponentProps<typeof AlertDialogPrimitive.Cancel>) {
  return (
    <AlertDialogPrimitive.Cancel
      className={cn(buttonVariants({ variant: "outline" }), className)}
      {...props}
    />
  )
}

export {
  AlertDialog,
  AlertDialogPortal,
  AlertDialogOverlay,
  AlertDialogTrigger,
  AlertDialogContent,
  AlertDialogHeader,
  AlertDialogFooter,
  AlertDialogTitle,
  AlertDialogDescription,
  AlertDialogAction,
  AlertDialogCancel,
}
````

## File: components/ui/alert.tsx
````typescript
import * as React from "react"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const alertVariants = cva(
  "relative w-full rounded-lg border px-4 py-3 text-sm grid has-[>svg]:grid-cols-[calc(var(--spacing)*4)_1fr] grid-cols-[0_1fr] has-[>svg]:gap-x-3 gap-y-0.5 items-start [&>svg]:size-4 [&>svg]:translate-y-0.5 [&>svg]:text-current",
  {
    variants: {
      variant: {
        default: "bg-card text-card-foreground",
        destructive:
          "text-destructive bg-card [&>svg]:text-current *:data-[slot=alert-description]:text-destructive/90",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
)

function Alert({
  className,
  variant,
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof alertVariants>) {
  return (
    <div
      data-slot="alert"
      role="alert"
      className={cn(alertVariants({ variant }), className)}
      {...props}
    />
  )
}

function AlertTitle({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="alert-title"
      className={cn(
        "col-start-2 line-clamp-1 min-h-4 font-medium tracking-tight",
        className
      )}
      {...props}
    />
  )
}

function AlertDescription({
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="alert-description"
      className={cn(
        "text-muted-foreground col-start-2 grid justify-items-start gap-1 text-sm [&_p]:leading-relaxed",
        className
      )}
      {...props}
    />
  )
}

export { Alert, AlertTitle, AlertDescription }
````

## File: components/ui/aspect-ratio.tsx
````typescript
"use client"

import { AspectRatio as AspectRatioPrimitive } from "radix-ui"

function AspectRatio({
  ...props
}: React.ComponentProps<typeof AspectRatioPrimitive.Root>) {
  return <AspectRatioPrimitive.Root data-slot="aspect-ratio" {...props} />
}

export { AspectRatio }
````

## File: components/ui/avatar.tsx
````typescript
"use client"

import * as React from "react"
import { Avatar as AvatarPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Avatar({
  className,
  ...props
}: React.ComponentProps<typeof AvatarPrimitive.Root>) {
  return (
    <AvatarPrimitive.Root
      data-slot="avatar"
      className={cn(
        "relative flex size-8 shrink-0 overflow-hidden rounded-full",
        className
      )}
      {...props}
    />
  )
}

function AvatarImage({
  className,
  ...props
}: React.ComponentProps<typeof AvatarPrimitive.Image>) {
  return (
    <AvatarPrimitive.Image
      data-slot="avatar-image"
      className={cn("aspect-square size-full", className)}
      {...props}
    />
  )
}

function AvatarFallback({
  className,
  ...props
}: React.ComponentProps<typeof AvatarPrimitive.Fallback>) {
  return (
    <AvatarPrimitive.Fallback
      data-slot="avatar-fallback"
      className={cn(
        "bg-muted flex size-full items-center justify-center rounded-full",
        className
      )}
      {...props}
    />
  )
}

export { Avatar, AvatarImage, AvatarFallback }
````

## File: components/ui/badge.tsx
````typescript
import * as React from "react"
import { Slot as SlotPrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const badgeVariants = cva(
  "inline-flex items-center justify-center rounded-full border px-2 py-0.5 text-xs font-medium w-fit whitespace-nowrap shrink-0 [&>svg]:size-3 gap-1 [&>svg]:pointer-events-none focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px] aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive transition-[color,box-shadow] overflow-hidden",
  {
    variants: {
      variant: {
        default:
          "border-transparent bg-primary text-primary-foreground [a&]:hover:bg-primary/90",
        secondary:
          "border-transparent bg-secondary text-secondary-foreground [a&]:hover:bg-secondary/90",
        destructive:
          "border-transparent bg-destructive text-white [a&]:hover:bg-destructive/90 focus-visible:ring-destructive/20 dark:focus-visible:ring-destructive/40 dark:bg-destructive/60",
        outline:
          "text-foreground [a&]:hover:bg-accent [a&]:hover:text-accent-foreground",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
)

function Badge({
  className,
  variant,
  asChild = false,
  ...props
}: React.ComponentProps<"span"> &
  VariantProps<typeof badgeVariants> & { asChild?: boolean }) {
  const Comp = asChild ? SlotPrimitive.Slot : "span"

  return (
    <Comp
      data-slot="badge"
      className={cn(badgeVariants({ variant }), className)}
      {...props}
    />
  )
}

export { Badge, badgeVariants }
````

## File: components/ui/breadcrumb.tsx
````typescript
import * as React from "react"
import { Slot as SlotPrimitive } from "radix-ui"
import { ChevronRight, MoreHorizontal } from "lucide-react"

import { cn } from "@/lib/utils"

function Breadcrumb({ ...props }: React.ComponentProps<"nav">) {
  return <nav aria-label="breadcrumb" data-slot="breadcrumb" {...props} />
}

function BreadcrumbList({ className, ...props }: React.ComponentProps<"ol">) {
  return (
    <ol
      data-slot="breadcrumb-list"
      className={cn(
        "text-muted-foreground flex flex-wrap items-center gap-1.5 text-sm break-words sm:gap-2.5",
        className
      )}
      {...props}
    />
  )
}

function BreadcrumbItem({ className, ...props }: React.ComponentProps<"li">) {
  return (
    <li
      data-slot="breadcrumb-item"
      className={cn("inline-flex items-center gap-1.5", className)}
      {...props}
    />
  )
}

function BreadcrumbLink({
  asChild,
  className,
  ...props
}: React.ComponentProps<"a"> & {
  asChild?: boolean
}) {
  const Comp = asChild ? SlotPrimitive.Slot : "a"

  return (
    <Comp
      data-slot="breadcrumb-link"
      className={cn("hover:text-foreground transition-colors", className)}
      {...props}
    />
  )
}

function BreadcrumbPage({ className, ...props }: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="breadcrumb-page"
      role="link"
      aria-disabled="true"
      aria-current="page"
      className={cn("text-foreground font-normal", className)}
      {...props}
    />
  )
}

function BreadcrumbSeparator({
  children,
  className,
  ...props
}: React.ComponentProps<"li">) {
  return (
    <li
      data-slot="breadcrumb-separator"
      role="presentation"
      aria-hidden="true"
      className={cn("[&>svg]:size-3.5", className)}
      {...props}
    >
      {children ?? <ChevronRight />}
    </li>
  )
}

function BreadcrumbEllipsis({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="breadcrumb-ellipsis"
      role="presentation"
      aria-hidden="true"
      className={cn("flex size-9 items-center justify-center", className)}
      {...props}
    >
      <MoreHorizontal className="size-4" />
      <span className="sr-only">More</span>
    </span>
  )
}

export {
  Breadcrumb,
  BreadcrumbList,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbPage,
  BreadcrumbSeparator,
  BreadcrumbEllipsis,
}
````

## File: components/ui/button-group.tsx
````typescript
import { Slot as SlotPrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"
import { Separator } from "@/components/ui/separator"

const buttonGroupVariants = cva(
  "flex w-fit items-stretch [&>*]:focus-visible:z-10 [&>*]:focus-visible:relative [&>[data-slot=select-trigger]:not([class*='w-'])]:w-fit [&>input]:flex-1 has-[select[aria-hidden=true]:last-child]:[&>[data-slot=select-trigger]:last-of-type]:rounded-r-md has-[>[data-slot=button-group]]:gap-2",
  {
    variants: {
      orientation: {
        horizontal:
          "[&>*:not(:first-child)]:rounded-l-none [&>*:not(:first-child)]:border-l-0 [&>*:not(:last-child)]:rounded-r-none",
        vertical:
          "flex-col [&>*:not(:first-child)]:rounded-t-none [&>*:not(:first-child)]:border-t-0 [&>*:not(:last-child)]:rounded-b-none",
      },
    },
    defaultVariants: {
      orientation: "horizontal",
    },
  }
)

function ButtonGroup({
  className,
  orientation,
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof buttonGroupVariants>) {
  return (
    <div
      role="group"
      data-slot="button-group"
      data-orientation={orientation}
      className={cn(buttonGroupVariants({ orientation }), className)}
      {...props}
    />
  )
}

function ButtonGroupText({
  className,
  asChild = false,
  ...props
}: React.ComponentProps<"div"> & {
  asChild?: boolean
}) {
  const Comp = asChild ? SlotPrimitive.Slot : "div"

  return (
    <Comp
      className={cn(
        "bg-muted flex items-center gap-2 rounded-md border px-4 text-sm font-medium shadow-xs [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function ButtonGroupSeparator({
  className,
  orientation = "vertical",
  ...props
}: React.ComponentProps<typeof Separator>) {
  return (
    <Separator
      data-slot="button-group-separator"
      orientation={orientation}
      className={cn(
        "bg-input relative !m-0 self-stretch data-[orientation=vertical]:h-auto",
        className
      )}
      {...props}
    />
  )
}

export {
  ButtonGroup,
  ButtonGroupSeparator,
  ButtonGroupText,
  buttonGroupVariants,
}
````

## File: components/ui/button.tsx
````typescript
import * as React from "react"
import { Slot as SlotPrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50 [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4 shrink-0 [&_svg]:shrink-0 outline-none focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px] aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive:
          "bg-destructive text-white hover:bg-destructive/90 focus-visible:ring-destructive/20 dark:focus-visible:ring-destructive/40 dark:bg-destructive/60",
        outline:
          "border bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost:
          "hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2 has-[>svg]:px-3",
        sm: "h-8 rounded-md gap-1.5 px-3 has-[>svg]:px-2.5",
        lg: "h-10 rounded-md px-6 has-[>svg]:px-4",
        icon: "size-9",
        "icon-sm": "size-8",
        "icon-lg": "size-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function Button({
  className,
  variant,
  size,
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean
  }) {
  const Comp = asChild ? SlotPrimitive.Slot : "button"

  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Button, buttonVariants }
````

## File: components/ui/calendar.tsx
````typescript
"use client"

import * as React from "react"
import {
  ChevronDownIcon,
  ChevronLeftIcon,
  ChevronRightIcon,
} from "lucide-react"
import { DayButton, DayPicker, getDefaultClassNames } from "react-day-picker"

import { cn } from "@/lib/utils"
import { Button, buttonVariants } from "@/components/ui/button"

function Calendar({
  className,
  classNames,
  showOutsideDays = true,
  captionLayout = "label",
  buttonVariant = "ghost",
  formatters,
  components,
  ...props
}: React.ComponentProps<typeof DayPicker> & {
  buttonVariant?: React.ComponentProps<typeof Button>["variant"]
}) {
  const defaultClassNames = getDefaultClassNames()

  return (
    <DayPicker
      showOutsideDays={showOutsideDays}
      className={cn(
        "bg-background group/calendar p-3 [--cell-size:--spacing(8)] [[data-slot=card-content]_&]:bg-transparent [[data-slot=popover-content]_&]:bg-transparent",
        String.raw`rtl:**:[.rdp-button\_next>svg]:rotate-180`,
        String.raw`rtl:**:[.rdp-button\_previous>svg]:rotate-180`,
        className
      )}
      captionLayout={captionLayout}
      formatters={{
        formatMonthDropdown: (date) =>
          date.toLocaleString("default", { month: "short" }),
        ...formatters,
      }}
      classNames={{
        root: cn("w-fit", defaultClassNames.root),
        months: cn(
          "flex gap-4 flex-col md:flex-row relative",
          defaultClassNames.months
        ),
        month: cn("flex flex-col w-full gap-4", defaultClassNames.month),
        nav: cn(
          "flex items-center gap-1 w-full absolute top-0 inset-x-0 justify-between",
          defaultClassNames.nav
        ),
        button_previous: cn(
          buttonVariants({ variant: buttonVariant }),
          "size-(--cell-size) aria-disabled:opacity-50 p-0 select-none",
          defaultClassNames.button_previous
        ),
        button_next: cn(
          buttonVariants({ variant: buttonVariant }),
          "size-(--cell-size) aria-disabled:opacity-50 p-0 select-none",
          defaultClassNames.button_next
        ),
        month_caption: cn(
          "flex items-center justify-center h-(--cell-size) w-full px-(--cell-size)",
          defaultClassNames.month_caption
        ),
        dropdowns: cn(
          "w-full flex items-center text-sm font-medium justify-center h-(--cell-size) gap-1.5",
          defaultClassNames.dropdowns
        ),
        dropdown_root: cn(
          "relative has-focus:border-ring border border-input shadow-xs has-focus:ring-ring/50 has-focus:ring-[3px] rounded-md",
          defaultClassNames.dropdown_root
        ),
        dropdown: cn(
          "absolute bg-popover inset-0 opacity-0",
          defaultClassNames.dropdown
        ),
        caption_label: cn(
          "select-none font-medium",
          captionLayout === "label"
            ? "text-sm"
            : "rounded-md pl-2 pr-1 flex items-center gap-1 text-sm h-8 [&>svg]:text-muted-foreground [&>svg]:size-3.5",
          defaultClassNames.caption_label
        ),
        table: "w-full border-collapse",
        weekdays: cn("flex", defaultClassNames.weekdays),
        weekday: cn(
          "text-muted-foreground rounded-md flex-1 font-normal text-[0.8rem] select-none",
          defaultClassNames.weekday
        ),
        week: cn("flex w-full mt-2", defaultClassNames.week),
        week_number_header: cn(
          "select-none w-(--cell-size)",
          defaultClassNames.week_number_header
        ),
        week_number: cn(
          "text-[0.8rem] select-none text-muted-foreground",
          defaultClassNames.week_number
        ),
        day: cn(
          "relative w-full h-full p-0 text-center [&:last-child[data-selected=true]_button]:rounded-r-md group/day aspect-square select-none",
          props.showWeekNumber
            ? "[&:nth-child(2)[data-selected=true]_button]:rounded-l-md"
            : "[&:first-child[data-selected=true]_button]:rounded-l-md",
          defaultClassNames.day
        ),
        range_start: cn(
          "rounded-l-md bg-accent",
          defaultClassNames.range_start
        ),
        range_middle: cn("rounded-none", defaultClassNames.range_middle),
        range_end: cn("rounded-r-md bg-accent", defaultClassNames.range_end),
        today: cn(
          "bg-accent text-accent-foreground rounded-md data-[selected=true]:rounded-none",
          defaultClassNames.today
        ),
        outside: cn(
          "text-muted-foreground aria-selected:text-muted-foreground",
          defaultClassNames.outside
        ),
        disabled: cn(
          "text-muted-foreground opacity-50",
          defaultClassNames.disabled
        ),
        hidden: cn("invisible", defaultClassNames.hidden),
        ...classNames,
      }}
      components={{
        Root: ({ className, rootRef, ...props }) => {
          return (
            <div
              data-slot="calendar"
              ref={rootRef}
              className={cn(className)}
              {...props}
            />
          )
        },
        Chevron: ({ className, orientation, ...props }) => {
          if (orientation === "left") {
            return (
              <ChevronLeftIcon className={cn("size-4", className)} {...props} />
            )
          }

          if (orientation === "right") {
            return (
              <ChevronRightIcon
                className={cn("size-4", className)}
                {...props}
              />
            )
          }

          return (
            <ChevronDownIcon className={cn("size-4", className)} {...props} />
          )
        },
        DayButton: CalendarDayButton,
        WeekNumber: ({ children, ...props }) => {
          return (
            <td {...props}>
              <div className="flex size-(--cell-size) items-center justify-center text-center">
                {children}
              </div>
            </td>
          )
        },
        ...components,
      }}
      {...props}
    />
  )
}

function CalendarDayButton({
  className,
  day,
  modifiers,
  ...props
}: React.ComponentProps<typeof DayButton>) {
  const defaultClassNames = getDefaultClassNames()

  const ref = React.useRef<HTMLButtonElement>(null)
  React.useEffect(() => {
    if (modifiers.focused) ref.current?.focus()
  }, [modifiers.focused])

  return (
    <Button
      ref={ref}
      variant="ghost"
      size="icon"
      data-day={day.date.toLocaleDateString()}
      data-selected-single={
        modifiers.selected &&
        !modifiers.range_start &&
        !modifiers.range_end &&
        !modifiers.range_middle
      }
      data-range-start={modifiers.range_start}
      data-range-end={modifiers.range_end}
      data-range-middle={modifiers.range_middle}
      className={cn(
        "data-[selected-single=true]:bg-primary data-[selected-single=true]:text-primary-foreground data-[range-middle=true]:bg-accent data-[range-middle=true]:text-accent-foreground data-[range-start=true]:bg-primary data-[range-start=true]:text-primary-foreground data-[range-end=true]:bg-primary data-[range-end=true]:text-primary-foreground group-data-[focused=true]/day:border-ring group-data-[focused=true]/day:ring-ring/50 dark:hover:text-accent-foreground flex aspect-square size-auto w-full min-w-(--cell-size) flex-col gap-1 leading-none font-normal group-data-[focused=true]/day:relative group-data-[focused=true]/day:z-10 group-data-[focused=true]/day:ring-[3px] data-[range-end=true]:rounded-md data-[range-end=true]:rounded-r-md data-[range-middle=true]:rounded-none data-[range-start=true]:rounded-md data-[range-start=true]:rounded-l-md [&>span]:text-xs [&>span]:opacity-70",
        defaultClassNames.day,
        className
      )}
      {...props}
    />
  )
}

export { Calendar, CalendarDayButton }
````

## File: components/ui/card.tsx
````typescript
import * as React from "react"

import { cn } from "@/lib/utils"

function Card({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card"
      className={cn(
        "bg-card text-card-foreground flex flex-col gap-6 rounded-xl border py-6 shadow-sm",
        className
      )}
      {...props}
    />
  )
}

function CardHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-header"
      className={cn(
        "@container/card-header grid auto-rows-min grid-rows-[auto_auto] items-start gap-2 px-6 has-data-[slot=card-action]:grid-cols-[1fr_auto] [.border-b]:pb-6",
        className
      )}
      {...props}
    />
  )
}

function CardTitle({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-title"
      className={cn("leading-none font-semibold", className)}
      {...props}
    />
  )
}

function CardDescription({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-description"
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

function CardAction({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-action"
      className={cn(
        "col-start-2 row-span-2 row-start-1 self-start justify-self-end",
        className
      )}
      {...props}
    />
  )
}

function CardContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-content"
      className={cn("px-6", className)}
      {...props}
    />
  )
}

function CardFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card-footer"
      className={cn("flex items-center px-6 [.border-t]:pt-6", className)}
      {...props}
    />
  )
}

export {
  Card,
  CardHeader,
  CardFooter,
  CardTitle,
  CardAction,
  CardDescription,
  CardContent,
}
````

## File: components/ui/carousel.tsx
````typescript
"use client"

import * as React from "react"
import useEmblaCarousel, {
  type UseEmblaCarouselType,
} from "embla-carousel-react"
import { ArrowLeft, ArrowRight } from "lucide-react"

import { cn } from "@/lib/utils"
import { Button } from "@/components/ui/button"

type CarouselApi = UseEmblaCarouselType[1]
type UseCarouselParameters = Parameters<typeof useEmblaCarousel>
type CarouselOptions = UseCarouselParameters[0]
type CarouselPlugin = UseCarouselParameters[1]

type CarouselProps = {
  opts?: CarouselOptions
  plugins?: CarouselPlugin
  orientation?: "horizontal" | "vertical"
  setApi?: (api: CarouselApi) => void
}

type CarouselContextProps = {
  carouselRef: ReturnType<typeof useEmblaCarousel>[0]
  api: ReturnType<typeof useEmblaCarousel>[1]
  scrollPrev: () => void
  scrollNext: () => void
  canScrollPrev: boolean
  canScrollNext: boolean
} & CarouselProps

const CarouselContext = React.createContext<CarouselContextProps | null>(null)

function useCarousel() {
  const context = React.useContext(CarouselContext)

  if (!context) {
    throw new Error("useCarousel must be used within a <Carousel />")
  }

  return context
}

function Carousel({
  orientation = "horizontal",
  opts,
  setApi,
  plugins,
  className,
  children,
  ...props
}: React.ComponentProps<"div"> & CarouselProps) {
  const [carouselRef, api] = useEmblaCarousel(
    {
      ...opts,
      axis: orientation === "horizontal" ? "x" : "y",
    },
    plugins
  )
  const [canScrollPrev, setCanScrollPrev] = React.useState(false)
  const [canScrollNext, setCanScrollNext] = React.useState(false)

  const onSelect = React.useCallback((api: CarouselApi) => {
    if (!api) return
    setCanScrollPrev(api.canScrollPrev())
    setCanScrollNext(api.canScrollNext())
  }, [])

  const scrollPrev = React.useCallback(() => {
    api?.scrollPrev()
  }, [api])

  const scrollNext = React.useCallback(() => {
    api?.scrollNext()
  }, [api])

  const handleKeyDown = React.useCallback(
    (event: React.KeyboardEvent<HTMLDivElement>) => {
      if (event.key === "ArrowLeft") {
        event.preventDefault()
        scrollPrev()
      } else if (event.key === "ArrowRight") {
        event.preventDefault()
        scrollNext()
      }
    },
    [scrollPrev, scrollNext]
  )

  React.useEffect(() => {
    if (!api || !setApi) return
    setApi(api)
  }, [api, setApi])

  React.useEffect(() => {
    if (!api) return
    onSelect(api)
    api.on("reInit", onSelect)
    api.on("select", onSelect)

    return () => {
      api?.off("select", onSelect)
    }
  }, [api, onSelect])

  return (
    <CarouselContext.Provider
      value={{
        carouselRef,
        api: api,
        opts,
        orientation:
          orientation || (opts?.axis === "y" ? "vertical" : "horizontal"),
        scrollPrev,
        scrollNext,
        canScrollPrev,
        canScrollNext,
      }}
    >
      <div
        onKeyDownCapture={handleKeyDown}
        className={cn("relative", className)}
        role="region"
        aria-roledescription="carousel"
        data-slot="carousel"
        {...props}
      >
        {children}
      </div>
    </CarouselContext.Provider>
  )
}

function CarouselContent({ className, ...props }: React.ComponentProps<"div">) {
  const { carouselRef, orientation } = useCarousel()

  return (
    <div
      ref={carouselRef}
      className="overflow-hidden"
      data-slot="carousel-content"
    >
      <div
        className={cn(
          "flex",
          orientation === "horizontal" ? "-ml-4" : "-mt-4 flex-col",
          className
        )}
        {...props}
      />
    </div>
  )
}

function CarouselItem({ className, ...props }: React.ComponentProps<"div">) {
  const { orientation } = useCarousel()

  return (
    <div
      role="group"
      aria-roledescription="slide"
      data-slot="carousel-item"
      className={cn(
        "min-w-0 shrink-0 grow-0 basis-full",
        orientation === "horizontal" ? "pl-4" : "pt-4",
        className
      )}
      {...props}
    />
  )
}

function CarouselPrevious({
  className,
  variant = "outline",
  size = "icon",
  ...props
}: React.ComponentProps<typeof Button>) {
  const { orientation, scrollPrev, canScrollPrev } = useCarousel()

  return (
    <Button
      data-slot="carousel-previous"
      variant={variant}
      size={size}
      className={cn(
        "absolute size-8 rounded-full",
        orientation === "horizontal"
          ? "top-1/2 -left-12 -translate-y-1/2"
          : "-top-12 left-1/2 -translate-x-1/2 rotate-90",
        className
      )}
      disabled={!canScrollPrev}
      onClick={scrollPrev}
      {...props}
    >
      <ArrowLeft />
      <span className="sr-only">Previous slide</span>
    </Button>
  )
}

function CarouselNext({
  className,
  variant = "outline",
  size = "icon",
  ...props
}: React.ComponentProps<typeof Button>) {
  const { orientation, scrollNext, canScrollNext } = useCarousel()

  return (
    <Button
      data-slot="carousel-next"
      variant={variant}
      size={size}
      className={cn(
        "absolute size-8 rounded-full",
        orientation === "horizontal"
          ? "top-1/2 -right-12 -translate-y-1/2"
          : "-bottom-12 left-1/2 -translate-x-1/2 rotate-90",
        className
      )}
      disabled={!canScrollNext}
      onClick={scrollNext}
      {...props}
    >
      <ArrowRight />
      <span className="sr-only">Next slide</span>
    </Button>
  )
}

export {
  type CarouselApi,
  Carousel,
  CarouselContent,
  CarouselItem,
  CarouselPrevious,
  CarouselNext,
}
````

## File: components/ui/chart.tsx
````typescript
"use client"

import * as React from "react"
import * as RechartsPrimitive from "recharts"

import { cn } from "@/lib/utils"

// Format: { THEME_NAME: CSS_SELECTOR }
const THEMES = { light: "", dark: ".dark" } as const

export type ChartConfig = {
  [k in string]: {
    label?: React.ReactNode
    icon?: React.ComponentType
  } & (
    | { color?: string; theme?: never }
    | { color?: never; theme: Record<keyof typeof THEMES, string> }
  )
}

type ChartContextProps = {
  config: ChartConfig
}

const ChartContext = React.createContext<ChartContextProps | null>(null)

function useChart() {
  const context = React.useContext(ChartContext)

  if (!context) {
    throw new Error("useChart must be used within a <ChartContainer />")
  }

  return context
}

function ChartContainer({
  id,
  className,
  children,
  config,
  ...props
}: React.ComponentProps<"div"> & {
  config: ChartConfig
  children: React.ComponentProps<
    typeof RechartsPrimitive.ResponsiveContainer
  >["children"]
}) {
  const uniqueId = React.useId()
  const chartId = `chart-${id || uniqueId.replace(/:/g, "")}`

  return (
    <ChartContext.Provider value={{ config }}>
      <div
        data-slot="chart"
        data-chart={chartId}
        className={cn(
          "[&_.recharts-cartesian-axis-tick_text]:fill-muted-foreground [&_.recharts-cartesian-grid_line[stroke='#ccc']]:stroke-border/50 [&_.recharts-curve.recharts-tooltip-cursor]:stroke-border [&_.recharts-polar-grid_[stroke='#ccc']]:stroke-border [&_.recharts-radial-bar-background-sector]:fill-muted [&_.recharts-rectangle.recharts-tooltip-cursor]:fill-muted [&_.recharts-reference-line_[stroke='#ccc']]:stroke-border flex aspect-video justify-center text-xs [&_.recharts-dot[stroke='#fff']]:stroke-transparent [&_.recharts-layer]:outline-hidden [&_.recharts-sector]:outline-hidden [&_.recharts-sector[stroke='#fff']]:stroke-transparent [&_.recharts-surface]:outline-hidden",
          className
        )}
        {...props}
      >
        <ChartStyle id={chartId} config={config} />
        <RechartsPrimitive.ResponsiveContainer>
          {children}
        </RechartsPrimitive.ResponsiveContainer>
      </div>
    </ChartContext.Provider>
  )
}

const ChartStyle = ({ id, config }: { id: string; config: ChartConfig }) => {
  const colorConfig = Object.entries(config).filter(
    ([, config]) => config.theme || config.color
  )

  if (!colorConfig.length) {
    return null
  }

  return (
    <style
      dangerouslySetInnerHTML={{
        __html: Object.entries(THEMES)
          .map(
            ([theme, prefix]) => `
${prefix} [data-chart=${id}] {
${colorConfig
  .map(([key, itemConfig]) => {
    const color =
      itemConfig.theme?.[theme as keyof typeof itemConfig.theme] ||
      itemConfig.color
    return color ? `  --color-${key}: ${color};` : null
  })
  .join("\n")}
}
`
          )
          .join("\n"),
      }}
    />
  )
}

const ChartTooltip = RechartsPrimitive.Tooltip

function ChartTooltipContent({
  active,
  payload,
  className,
  indicator = "dot",
  hideLabel = false,
  hideIndicator = false,
  label,
  labelFormatter,
  labelClassName,
  formatter,
  valueFormatter,
  footer,
  color,
  nameKey,
  labelKey,
}: React.ComponentProps<typeof RechartsPrimitive.Tooltip> &
  React.ComponentProps<"div"> & {
    hideLabel?: boolean
    hideIndicator?: boolean
    indicator?: "line" | "dot" | "dashed"
    nameKey?: string
    labelKey?: string
    valueFormatter?: (value: number | string) => string
    footer?: React.ReactNode
  }) {
  const { config } = useChart()

  const tooltipLabel = React.useMemo(() => {
    if (hideLabel || !payload?.length) {
      return null
    }

    const [item] = payload
    const key = `${labelKey || item?.dataKey || item?.name || "value"}`
    const itemConfig = getPayloadConfigFromPayload(config, item, key)
    const value =
      !labelKey && typeof label === "string"
        ? config[label as keyof typeof config]?.label || label
        : itemConfig?.label

    if (labelFormatter) {
      return (
        <div className={cn("font-medium", labelClassName)}>
          {labelFormatter(value, payload)}
        </div>
      )
    }

    if (!value) {
      return null
    }

    return <div className={cn("font-medium", labelClassName)}>{value}</div>
  }, [
    label,
    labelFormatter,
    payload,
    hideLabel,
    labelClassName,
    config,
    labelKey,
  ])

  if (!active || !payload?.length) {
    return null
  }

  const nestLabel = payload.length === 1 && indicator !== "dot"

  return (
    <div
      className={cn(
        "border-border/50 bg-background grid min-w-[8rem] items-start gap-1.5 rounded-lg border px-2.5 py-1.5 text-xs shadow-xl",
        className
      )}
    >
      {!nestLabel ? tooltipLabel : null}
      <div className="grid gap-1.5">
        {payload
          .filter((item) => item.type !== "none")
          .map((item, index) => {
            const key = `${nameKey || item.name || item.dataKey || "value"}`
            const itemConfig = getPayloadConfigFromPayload(config, item, key)
            const indicatorColor = color || item.payload.fill || item.color

            return (
              <div
                key={item.dataKey}
                className={cn(
                  "[&>svg]:text-muted-foreground flex w-full flex-wrap items-stretch gap-2 [&>svg]:h-2.5 [&>svg]:w-2.5",
                  indicator === "dot" && "items-center"
                )}
              >
                {formatter && item?.value !== undefined && item.name ? (
                  formatter(item.value, item.name, item, index, item.payload)
                ) : (
                  <>
                    {itemConfig?.icon ? (
                      <itemConfig.icon />
                    ) : (
                      !hideIndicator && (
                        <div
                          className={cn(
                            "shrink-0 rounded-[2px] border-(--color-border) bg-(--color-bg)",
                            {
                              "h-2.5 w-2.5": indicator === "dot",
                              "w-1": indicator === "line",
                              "w-0 border-[1.5px] border-dashed bg-transparent":
                                indicator === "dashed",
                              "my-0.5": nestLabel && indicator === "dashed",
                            }
                          )}
                          style={
                            {
                              "--color-bg": indicatorColor,
                              "--color-border": indicatorColor,
                            } as React.CSSProperties
                          }
                        />
                      )
                    )}
                    <div
                      className={cn(
                        "flex flex-1 justify-between leading-none",
                        nestLabel ? "items-end" : "items-center"
                      )}
                    >
                      <div className="grid gap-1.5">
                        {nestLabel ? tooltipLabel : null}
                        <span className="text-muted-foreground">
                          {itemConfig?.label || item.name}
                        </span>
                      </div>
                      {item.value != null && (
                        <span className="text-foreground font-mono font-medium tabular-nums">
                          {valueFormatter && !Array.isArray(item.value)
                            ? valueFormatter(item.value)
                            : item.value.toLocaleString()}
                        </span>
                      )}
                    </div>
                  </>
                )}
              </div>
            )
          })}
      </div>
      {footer}
    </div>
  )
}

const ChartLegend = RechartsPrimitive.Legend

function ChartLegendContent({
  className,
  hideIcon = false,
  payload,
  verticalAlign = "bottom",
  nameKey,
}: React.ComponentProps<"div"> &
  Pick<RechartsPrimitive.LegendProps, "payload" | "verticalAlign"> & {
    hideIcon?: boolean
    nameKey?: string
  }) {
  const { config } = useChart()

  if (!payload?.length) {
    return null
  }

  return (
    <div
      className={cn(
        "flex items-center justify-center gap-4",
        verticalAlign === "top" ? "pb-3" : "pt-3",
        className
      )}
    >
      {payload
        .filter((item) => item.type !== "none")
        .map((item) => {
          const key = `${nameKey || item.dataKey || "value"}`
          const itemConfig = getPayloadConfigFromPayload(config, item, key)

          return (
            <div
              key={item.value}
              className={cn(
                "[&>svg]:text-muted-foreground flex items-center gap-1.5 [&>svg]:h-3 [&>svg]:w-3"
              )}
            >
              {itemConfig?.icon && !hideIcon ? (
                <itemConfig.icon />
              ) : (
                <div
                  className="h-2 w-2 shrink-0 rounded-[2px]"
                  style={{
                    backgroundColor: item.color,
                  }}
                />
              )}
              {itemConfig?.label}
            </div>
          )
        })}
    </div>
  )
}

// Helper to extract item config from a payload.
function getPayloadConfigFromPayload(
  config: ChartConfig,
  payload: unknown,
  key: string
) {
  if (typeof payload !== "object" || payload === null) {
    return undefined
  }

  const payloadPayload =
    "payload" in payload &&
    typeof payload.payload === "object" &&
    payload.payload !== null
      ? payload.payload
      : undefined

  let configLabelKey: string = key

  if (
    key in payload &&
    typeof payload[key as keyof typeof payload] === "string"
  ) {
    configLabelKey = payload[key as keyof typeof payload] as string
  } else if (
    payloadPayload &&
    key in payloadPayload &&
    typeof payloadPayload[key as keyof typeof payloadPayload] === "string"
  ) {
    configLabelKey = payloadPayload[
      key as keyof typeof payloadPayload
    ] as string
  }

  return configLabelKey in config
    ? config[configLabelKey]
    : config[key as keyof typeof config]
}

export {
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
  ChartLegend,
  ChartLegendContent,
  ChartStyle,
}
````

## File: components/ui/checkbox.tsx
````typescript
"use client"

import * as React from "react"
import { Checkbox as CheckboxPrimitive } from "radix-ui"
import { CheckIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Checkbox({
  className,
  ...props
}: React.ComponentProps<typeof CheckboxPrimitive.Root>) {
  return (
    <CheckboxPrimitive.Root
      data-slot="checkbox"
      className={cn(
        "peer border-input dark:bg-input/30 data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground dark:data-[state=checked]:bg-primary data-[state=checked]:border-primary focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive size-4 shrink-0 rounded-[4px] border shadow-xs transition-shadow outline-none focus-visible:ring-[3px] disabled:cursor-not-allowed disabled:opacity-50",
        className
      )}
      {...props}
    >
      <CheckboxPrimitive.Indicator
        data-slot="checkbox-indicator"
        className="grid place-content-center text-current transition-none"
      >
        <CheckIcon className="size-3.5" />
      </CheckboxPrimitive.Indicator>
    </CheckboxPrimitive.Root>
  )
}

export { Checkbox }
````

## File: components/ui/collapsible.tsx
````typescript
"use client"

import { Collapsible as CollapsiblePrimitive } from "radix-ui"

function Collapsible({
  ...props
}: React.ComponentProps<typeof CollapsiblePrimitive.Root>) {
  return <CollapsiblePrimitive.Root data-slot="collapsible" {...props} />
}

function CollapsibleTrigger({
  ...props
}: React.ComponentProps<typeof CollapsiblePrimitive.CollapsibleTrigger>) {
  return (
    <CollapsiblePrimitive.CollapsibleTrigger
      data-slot="collapsible-trigger"
      {...props}
    />
  )
}

function CollapsibleContent({
  ...props
}: React.ComponentProps<typeof CollapsiblePrimitive.CollapsibleContent>) {
  return (
    <CollapsiblePrimitive.CollapsibleContent
      data-slot="collapsible-content"
      {...props}
    />
  )
}

export { Collapsible, CollapsibleTrigger, CollapsibleContent }
````

## File: components/ui/command.tsx
````typescript
"use client"

import * as React from "react"
import { Command as CommandPrimitive } from "cmdk"
import { SearchIcon } from "lucide-react"

import { cn } from "@/lib/utils"
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog"

function Command({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive>) {
  return (
    <CommandPrimitive
      data-slot="command"
      className={cn(
        "bg-popover text-popover-foreground flex h-full w-full flex-col overflow-hidden rounded-md",
        className
      )}
      {...props}
    />
  )
}

function CommandDialog({
  title = "Command Palette",
  description = "Search for a command to run...",
  children,
  className,
  showCloseButton = true,
  ...props
}: React.ComponentProps<typeof Dialog> & {
  title?: string
  description?: string
  className?: string
  showCloseButton?: boolean
}) {
  return (
    <Dialog {...props}>
      <DialogHeader className="sr-only">
        <DialogTitle>{title}</DialogTitle>
        <DialogDescription>{description}</DialogDescription>
      </DialogHeader>
      <DialogContent
        className={cn("overflow-hidden p-0", className)}
        showCloseButton={showCloseButton}
      >
        <Command className="[&_[cmdk-group-heading]]:text-muted-foreground **:data-[slot=command-input-wrapper]:h-12 [&_[cmdk-group-heading]]:px-2 [&_[cmdk-group-heading]]:font-medium [&_[cmdk-group]]:px-2 [&_[cmdk-group]:not([hidden])_~[cmdk-group]]:pt-0 [&_[cmdk-input-wrapper]_svg]:h-5 [&_[cmdk-input-wrapper]_svg]:w-5 [&_[cmdk-input]]:h-12 [&_[cmdk-item]]:px-2 [&_[cmdk-item]]:py-3 [&_[cmdk-item]_svg]:h-5 [&_[cmdk-item]_svg]:w-5">
          {children}
        </Command>
      </DialogContent>
    </Dialog>
  )
}

function CommandInput({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive.Input>) {
  return (
    <div
      data-slot="command-input-wrapper"
      className="flex h-9 items-center gap-2 border-b px-3"
    >
      <SearchIcon className="size-4 shrink-0 opacity-50" />
      <CommandPrimitive.Input
        data-slot="command-input"
        className={cn(
          "placeholder:text-muted-foreground flex h-10 w-full rounded-md bg-transparent py-3 text-sm outline-hidden disabled:cursor-not-allowed disabled:opacity-50",
          className
        )}
        {...props}
      />
    </div>
  )
}

function CommandList({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive.List>) {
  return (
    <CommandPrimitive.List
      data-slot="command-list"
      className={cn(
        "max-h-[300px] scroll-py-1 overflow-x-hidden overflow-y-auto",
        className
      )}
      {...props}
    />
  )
}

function CommandEmpty({
  ...props
}: React.ComponentProps<typeof CommandPrimitive.Empty>) {
  return (
    <CommandPrimitive.Empty
      data-slot="command-empty"
      className="py-6 text-center text-sm"
      {...props}
    />
  )
}

function CommandGroup({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive.Group>) {
  return (
    <CommandPrimitive.Group
      data-slot="command-group"
      className={cn(
        "text-foreground [&_[cmdk-group-heading]]:text-muted-foreground overflow-hidden p-1 [&_[cmdk-group-heading]]:px-2 [&_[cmdk-group-heading]]:py-1.5 [&_[cmdk-group-heading]]:text-xs [&_[cmdk-group-heading]]:font-medium",
        className
      )}
      {...props}
    />
  )
}

function CommandSeparator({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive.Separator>) {
  return (
    <CommandPrimitive.Separator
      data-slot="command-separator"
      className={cn("bg-border -mx-1 h-px", className)}
      {...props}
    />
  )
}

function CommandItem({
  className,
  ...props
}: React.ComponentProps<typeof CommandPrimitive.Item>) {
  return (
    <CommandPrimitive.Item
      data-slot="command-item"
      className={cn(
        "data-[selected=true]:bg-accent data-[selected=true]:text-accent-foreground [&_svg:not([class*='text-'])]:text-muted-foreground relative flex cursor-default items-center gap-2 rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[disabled=true]:pointer-events-none data-[disabled=true]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function CommandShortcut({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="command-shortcut"
      className={cn(
        "text-muted-foreground ml-auto text-xs tracking-widest",
        className
      )}
      {...props}
    />
  )
}

export {
  Command,
  CommandDialog,
  CommandInput,
  CommandList,
  CommandEmpty,
  CommandGroup,
  CommandItem,
  CommandShortcut,
  CommandSeparator,
}
````

## File: components/ui/context-menu.tsx
````typescript
"use client"

import * as React from "react"
import { ContextMenu as ContextMenuPrimitive } from "radix-ui"
import { CheckIcon, ChevronRightIcon, CircleIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function ContextMenu({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Root>) {
  return <ContextMenuPrimitive.Root data-slot="context-menu" {...props} />
}

function ContextMenuTrigger({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Trigger>) {
  return (
    <ContextMenuPrimitive.Trigger data-slot="context-menu-trigger" {...props} />
  )
}

function ContextMenuGroup({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Group>) {
  return (
    <ContextMenuPrimitive.Group data-slot="context-menu-group" {...props} />
  )
}

function ContextMenuPortal({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Portal>) {
  return (
    <ContextMenuPrimitive.Portal data-slot="context-menu-portal" {...props} />
  )
}

function ContextMenuSub({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Sub>) {
  return <ContextMenuPrimitive.Sub data-slot="context-menu-sub" {...props} />
}

function ContextMenuRadioGroup({
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.RadioGroup>) {
  return (
    <ContextMenuPrimitive.RadioGroup
      data-slot="context-menu-radio-group"
      {...props}
    />
  )
}

function ContextMenuSubTrigger({
  className,
  inset,
  children,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.SubTrigger> & {
  inset?: boolean
}) {
  return (
    <ContextMenuPrimitive.SubTrigger
      data-slot="context-menu-sub-trigger"
      data-inset={inset}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[state=open]:bg-accent data-[state=open]:text-accent-foreground [&_svg:not([class*='text-'])]:text-muted-foreground flex cursor-default items-center rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[inset]:pl-8 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      {children}
      <ChevronRightIcon className="ml-auto" />
    </ContextMenuPrimitive.SubTrigger>
  )
}

function ContextMenuSubContent({
  className,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.SubContent>) {
  return (
    <ContextMenuPrimitive.SubContent
      data-slot="context-menu-sub-content"
      className={cn(
        "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 min-w-[8rem] origin-(--radix-context-menu-content-transform-origin) overflow-hidden rounded-md border p-1 shadow-lg",
        className
      )}
      {...props}
    />
  )
}

function ContextMenuContent({
  className,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Content>) {
  return (
    <ContextMenuPrimitive.Portal>
      <ContextMenuPrimitive.Content
        data-slot="context-menu-content"
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 max-h-(--radix-context-menu-content-available-height) min-w-[8rem] origin-(--radix-context-menu-content-transform-origin) overflow-x-hidden overflow-y-auto rounded-md border p-1 shadow-md",
          className
        )}
        {...props}
      />
    </ContextMenuPrimitive.Portal>
  )
}

function ContextMenuItem({
  className,
  inset,
  variant = "default",
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Item> & {
  inset?: boolean
  variant?: "default" | "destructive"
}) {
  return (
    <ContextMenuPrimitive.Item
      data-slot="context-menu-item"
      data-inset={inset}
      data-variant={variant}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[variant=destructive]:text-destructive data-[variant=destructive]:focus:bg-destructive/10 dark:data-[variant=destructive]:focus:bg-destructive/20 data-[variant=destructive]:focus:text-destructive data-[variant=destructive]:*:[svg]:!text-destructive [&_svg:not([class*='text-'])]:text-muted-foreground relative flex cursor-default items-center gap-2 rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 data-[inset]:pl-8 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function ContextMenuCheckboxItem({
  className,
  children,
  checked,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.CheckboxItem>) {
  return (
    <ContextMenuPrimitive.CheckboxItem
      data-slot="context-menu-checkbox-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-sm py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      checked={checked}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <ContextMenuPrimitive.ItemIndicator>
          <CheckIcon className="size-4" />
        </ContextMenuPrimitive.ItemIndicator>
      </span>
      {children}
    </ContextMenuPrimitive.CheckboxItem>
  )
}

function ContextMenuRadioItem({
  className,
  children,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.RadioItem>) {
  return (
    <ContextMenuPrimitive.RadioItem
      data-slot="context-menu-radio-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-sm py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <ContextMenuPrimitive.ItemIndicator>
          <CircleIcon className="size-2 fill-current" />
        </ContextMenuPrimitive.ItemIndicator>
      </span>
      {children}
    </ContextMenuPrimitive.RadioItem>
  )
}

function ContextMenuLabel({
  className,
  inset,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Label> & {
  inset?: boolean
}) {
  return (
    <ContextMenuPrimitive.Label
      data-slot="context-menu-label"
      data-inset={inset}
      className={cn(
        "text-foreground px-2 py-1.5 text-sm font-medium data-[inset]:pl-8",
        className
      )}
      {...props}
    />
  )
}

function ContextMenuSeparator({
  className,
  ...props
}: React.ComponentProps<typeof ContextMenuPrimitive.Separator>) {
  return (
    <ContextMenuPrimitive.Separator
      data-slot="context-menu-separator"
      className={cn("bg-border -mx-1 my-1 h-px", className)}
      {...props}
    />
  )
}

function ContextMenuShortcut({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="context-menu-shortcut"
      className={cn(
        "text-muted-foreground ml-auto text-xs tracking-widest",
        className
      )}
      {...props}
    />
  )
}

export {
  ContextMenu,
  ContextMenuTrigger,
  ContextMenuContent,
  ContextMenuItem,
  ContextMenuCheckboxItem,
  ContextMenuRadioItem,
  ContextMenuLabel,
  ContextMenuSeparator,
  ContextMenuShortcut,
  ContextMenuGroup,
  ContextMenuPortal,
  ContextMenuSub,
  ContextMenuSubContent,
  ContextMenuSubTrigger,
  ContextMenuRadioGroup,
}
````

## File: components/ui/dialog.tsx
````typescript
"use client"

import * as React from "react"
import { Dialog as DialogPrimitive } from "radix-ui"
import { XIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Dialog({
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Root>) {
  return <DialogPrimitive.Root data-slot="dialog" {...props} />
}

function DialogTrigger({
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Trigger>) {
  return <DialogPrimitive.Trigger data-slot="dialog-trigger" {...props} />
}

function DialogPortal({
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Portal>) {
  return <DialogPrimitive.Portal data-slot="dialog-portal" {...props} />
}

function DialogClose({
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Close>) {
  return <DialogPrimitive.Close data-slot="dialog-close" {...props} />
}

function DialogOverlay({
  className,
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Overlay>) {
  return (
    <DialogPrimitive.Overlay
      data-slot="dialog-overlay"
      className={cn(
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50",
        className
      )}
      {...props}
    />
  )
}

function DialogContent({
  className,
  children,
  showCloseButton = true,
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Content> & {
  showCloseButton?: boolean
}) {
  return (
    <DialogPortal data-slot="dialog-portal">
      <DialogOverlay />
      <DialogPrimitive.Content
        data-slot="dialog-content"
        className={cn(
          "bg-background data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 fixed top-[50%] left-[50%] z-50 grid w-full max-w-[calc(100%-2rem)] translate-x-[-50%] translate-y-[-50%] gap-4 rounded-lg border p-6 shadow-lg duration-200 sm:max-w-lg",
          className
        )}
        {...props}
      >
        {children}
        {showCloseButton && (
          <DialogPrimitive.Close
            data-slot="dialog-close"
            className="ring-offset-background focus:ring-ring data-[state=open]:bg-accent data-[state=open]:text-muted-foreground absolute top-4 right-4 rounded-xs opacity-70 transition-opacity hover:opacity-100 focus:ring-2 focus:ring-offset-2 focus:outline-hidden disabled:pointer-events-none [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4"
          >
            <XIcon />
            <span className="sr-only">Close</span>
          </DialogPrimitive.Close>
        )}
      </DialogPrimitive.Content>
    </DialogPortal>
  )
}

function DialogHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="dialog-header"
      className={cn("flex flex-col gap-2 text-center sm:text-left", className)}
      {...props}
    />
  )
}

function DialogFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="dialog-footer"
      className={cn(
        "flex flex-col-reverse gap-2 sm:flex-row sm:justify-end",
        className
      )}
      {...props}
    />
  )
}

function DialogTitle({
  className,
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Title>) {
  return (
    <DialogPrimitive.Title
      data-slot="dialog-title"
      className={cn("text-lg leading-none font-semibold", className)}
      {...props}
    />
  )
}

function DialogDescription({
  className,
  ...props
}: React.ComponentProps<typeof DialogPrimitive.Description>) {
  return (
    <DialogPrimitive.Description
      data-slot="dialog-description"
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

export {
  Dialog,
  DialogClose,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogOverlay,
  DialogPortal,
  DialogTitle,
  DialogTrigger,
}
````

## File: components/ui/drawer.tsx
````typescript
"use client"

import * as React from "react"
import { Drawer as DrawerPrimitive } from "vaul"

import { cn } from "@/lib/utils"

function Drawer({
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Root>) {
  return <DrawerPrimitive.Root data-slot="drawer" {...props} />
}

function DrawerTrigger({
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Trigger>) {
  return <DrawerPrimitive.Trigger data-slot="drawer-trigger" {...props} />
}

function DrawerPortal({
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Portal>) {
  return <DrawerPrimitive.Portal data-slot="drawer-portal" {...props} />
}

function DrawerClose({
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Close>) {
  return <DrawerPrimitive.Close data-slot="drawer-close" {...props} />
}

function DrawerOverlay({
  className,
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Overlay>) {
  return (
    <DrawerPrimitive.Overlay
      data-slot="drawer-overlay"
      className={cn(
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50",
        className
      )}
      {...props}
    />
  )
}

function DrawerContent({
  className,
  children,
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Content>) {
  return (
    <DrawerPortal data-slot="drawer-portal">
      <DrawerOverlay />
      <DrawerPrimitive.Content
        data-slot="drawer-content"
        className={cn(
          "group/drawer-content bg-background fixed z-50 flex h-auto flex-col",
          "data-[vaul-drawer-direction=top]:inset-x-0 data-[vaul-drawer-direction=top]:top-0 data-[vaul-drawer-direction=top]:mb-24 data-[vaul-drawer-direction=top]:max-h-[80vh] data-[vaul-drawer-direction=top]:rounded-b-lg data-[vaul-drawer-direction=top]:border-b",
          "data-[vaul-drawer-direction=bottom]:inset-x-0 data-[vaul-drawer-direction=bottom]:bottom-0 data-[vaul-drawer-direction=bottom]:mt-24 data-[vaul-drawer-direction=bottom]:max-h-[80vh] data-[vaul-drawer-direction=bottom]:rounded-t-lg data-[vaul-drawer-direction=bottom]:border-t",
          "data-[vaul-drawer-direction=right]:inset-y-0 data-[vaul-drawer-direction=right]:right-0 data-[vaul-drawer-direction=right]:w-3/4 data-[vaul-drawer-direction=right]:border-l data-[vaul-drawer-direction=right]:sm:max-w-sm",
          "data-[vaul-drawer-direction=left]:inset-y-0 data-[vaul-drawer-direction=left]:left-0 data-[vaul-drawer-direction=left]:w-3/4 data-[vaul-drawer-direction=left]:border-r data-[vaul-drawer-direction=left]:sm:max-w-sm",
          className
        )}
        {...props}
      >
        <div className="bg-muted mx-auto mt-4 hidden h-2 w-[100px] shrink-0 rounded-full group-data-[vaul-drawer-direction=bottom]/drawer-content:block" />
        {children}
      </DrawerPrimitive.Content>
    </DrawerPortal>
  )
}

function DrawerHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="drawer-header"
      className={cn(
        "flex flex-col gap-0.5 p-4 group-data-[vaul-drawer-direction=bottom]/drawer-content:text-center group-data-[vaul-drawer-direction=top]/drawer-content:text-center md:gap-1.5 md:text-left",
        className
      )}
      {...props}
    />
  )
}

function DrawerFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="drawer-footer"
      className={cn("mt-auto flex flex-col gap-2 p-4", className)}
      {...props}
    />
  )
}

function DrawerTitle({
  className,
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Title>) {
  return (
    <DrawerPrimitive.Title
      data-slot="drawer-title"
      className={cn("text-foreground font-semibold", className)}
      {...props}
    />
  )
}

function DrawerDescription({
  className,
  ...props
}: React.ComponentProps<typeof DrawerPrimitive.Description>) {
  return (
    <DrawerPrimitive.Description
      data-slot="drawer-description"
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

export {
  Drawer,
  DrawerPortal,
  DrawerOverlay,
  DrawerTrigger,
  DrawerClose,
  DrawerContent,
  DrawerHeader,
  DrawerFooter,
  DrawerTitle,
  DrawerDescription,
}
````

## File: components/ui/dropdown-menu.tsx
````typescript
"use client"

import * as React from "react"
import { DropdownMenu as DropdownMenuPrimitive } from "radix-ui"
import { CheckIcon, ChevronRightIcon, CircleIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function DropdownMenu({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Root>) {
  return <DropdownMenuPrimitive.Root data-slot="dropdown-menu" {...props} />
}

function DropdownMenuPortal({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Portal>) {
  return (
    <DropdownMenuPrimitive.Portal data-slot="dropdown-menu-portal" {...props} />
  )
}

function DropdownMenuTrigger({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Trigger>) {
  return (
    <DropdownMenuPrimitive.Trigger
      data-slot="dropdown-menu-trigger"
      {...props}
    />
  )
}

function DropdownMenuContent({
  className,
  sideOffset = 4,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Content>) {
  return (
    <DropdownMenuPrimitive.Portal>
      <DropdownMenuPrimitive.Content
        data-slot="dropdown-menu-content"
        sideOffset={sideOffset}
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 max-h-(--radix-dropdown-menu-content-available-height) min-w-[8rem] origin-(--radix-dropdown-menu-content-transform-origin) overflow-x-hidden overflow-y-auto rounded-md border p-1 shadow-md",
          className
        )}
        {...props}
      />
    </DropdownMenuPrimitive.Portal>
  )
}

function DropdownMenuGroup({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Group>) {
  return (
    <DropdownMenuPrimitive.Group data-slot="dropdown-menu-group" {...props} />
  )
}

function DropdownMenuItem({
  className,
  inset,
  variant = "default",
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Item> & {
  inset?: boolean
  variant?: "default" | "destructive"
}) {
  return (
    <DropdownMenuPrimitive.Item
      data-slot="dropdown-menu-item"
      data-inset={inset}
      data-variant={variant}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[variant=destructive]:text-destructive data-[variant=destructive]:focus:bg-destructive/10 dark:data-[variant=destructive]:focus:bg-destructive/20 data-[variant=destructive]:focus:text-destructive data-[variant=destructive]:*:[svg]:!text-destructive [&_svg:not([class*='text-'])]:text-muted-foreground relative flex cursor-default items-center gap-2 rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 data-[inset]:pl-8 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function DropdownMenuCheckboxItem({
  className,
  children,
  checked,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.CheckboxItem>) {
  return (
    <DropdownMenuPrimitive.CheckboxItem
      data-slot="dropdown-menu-checkbox-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-sm py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      checked={checked}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <DropdownMenuPrimitive.ItemIndicator>
          <CheckIcon className="size-4" />
        </DropdownMenuPrimitive.ItemIndicator>
      </span>
      {children}
    </DropdownMenuPrimitive.CheckboxItem>
  )
}

function DropdownMenuRadioGroup({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.RadioGroup>) {
  return (
    <DropdownMenuPrimitive.RadioGroup
      data-slot="dropdown-menu-radio-group"
      {...props}
    />
  )
}

function DropdownMenuRadioItem({
  className,
  children,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.RadioItem>) {
  return (
    <DropdownMenuPrimitive.RadioItem
      data-slot="dropdown-menu-radio-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-sm py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <DropdownMenuPrimitive.ItemIndicator>
          <CircleIcon className="size-2 fill-current" />
        </DropdownMenuPrimitive.ItemIndicator>
      </span>
      {children}
    </DropdownMenuPrimitive.RadioItem>
  )
}

function DropdownMenuLabel({
  className,
  inset,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Label> & {
  inset?: boolean
}) {
  return (
    <DropdownMenuPrimitive.Label
      data-slot="dropdown-menu-label"
      data-inset={inset}
      className={cn(
        "px-2 py-1.5 text-sm font-medium data-[inset]:pl-8",
        className
      )}
      {...props}
    />
  )
}

function DropdownMenuSeparator({
  className,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Separator>) {
  return (
    <DropdownMenuPrimitive.Separator
      data-slot="dropdown-menu-separator"
      className={cn("bg-border -mx-1 my-1 h-px", className)}
      {...props}
    />
  )
}

function DropdownMenuShortcut({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="dropdown-menu-shortcut"
      className={cn(
        "text-muted-foreground ml-auto text-xs tracking-widest",
        className
      )}
      {...props}
    />
  )
}

function DropdownMenuSub({
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Sub>) {
  return <DropdownMenuPrimitive.Sub data-slot="dropdown-menu-sub" {...props} />
}

function DropdownMenuSubTrigger({
  className,
  inset,
  children,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.SubTrigger> & {
  inset?: boolean
}) {
  return (
    <DropdownMenuPrimitive.SubTrigger
      data-slot="dropdown-menu-sub-trigger"
      data-inset={inset}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[state=open]:bg-accent data-[state=open]:text-accent-foreground [&_svg:not([class*='text-'])]:text-muted-foreground flex cursor-default items-center gap-2 rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[inset]:pl-8 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      {children}
      <ChevronRightIcon className="ml-auto size-4" />
    </DropdownMenuPrimitive.SubTrigger>
  )
}

function DropdownMenuSubContent({
  className,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.SubContent>) {
  return (
    <DropdownMenuPrimitive.SubContent
      data-slot="dropdown-menu-sub-content"
      className={cn(
        "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 min-w-[8rem] origin-(--radix-dropdown-menu-content-transform-origin) overflow-hidden rounded-md border p-1 shadow-lg",
        className
      )}
      {...props}
    />
  )
}

export {
  DropdownMenu,
  DropdownMenuPortal,
  DropdownMenuTrigger,
  DropdownMenuContent,
  DropdownMenuGroup,
  DropdownMenuLabel,
  DropdownMenuItem,
  DropdownMenuCheckboxItem,
  DropdownMenuRadioGroup,
  DropdownMenuRadioItem,
  DropdownMenuSeparator,
  DropdownMenuShortcut,
  DropdownMenuSub,
  DropdownMenuSubTrigger,
  DropdownMenuSubContent,
}
````

## File: components/ui/empty.tsx
````typescript
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

function Empty({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="empty"
      className={cn(
        "flex min-w-0 flex-1 flex-col items-center justify-center gap-6 rounded-lg border-dashed p-6 text-center text-balance md:p-12",
        className
      )}
      {...props}
    />
  )
}

function EmptyHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="empty-header"
      className={cn(
        "flex max-w-sm flex-col items-center gap-2 text-center",
        className
      )}
      {...props}
    />
  )
}

const emptyMediaVariants = cva(
  "flex shrink-0 items-center justify-center mb-2 [&_svg]:pointer-events-none [&_svg]:shrink-0",
  {
    variants: {
      variant: {
        default: "bg-transparent",
        icon: "bg-muted text-foreground flex size-10 shrink-0 items-center justify-center rounded-lg [&_svg:not([class*='size-'])]:size-6",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
)

function EmptyMedia({
  className,
  variant = "default",
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof emptyMediaVariants>) {
  return (
    <div
      data-slot="empty-icon"
      data-variant={variant}
      className={cn(emptyMediaVariants({ variant, className }))}
      {...props}
    />
  )
}

function EmptyTitle({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="empty-title"
      className={cn("text-lg font-medium tracking-tight", className)}
      {...props}
    />
  )
}

function EmptyDescription({ className, ...props }: React.ComponentProps<"p">) {
  return (
    <div
      data-slot="empty-description"
      className={cn(
        "text-muted-foreground [&>a:hover]:text-primary text-sm/relaxed [&>a]:underline [&>a]:underline-offset-4",
        className
      )}
      {...props}
    />
  )
}

function EmptyContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="empty-content"
      className={cn(
        "flex w-full max-w-sm min-w-0 flex-col items-center gap-4 text-sm text-balance",
        className
      )}
      {...props}
    />
  )
}

export {
  Empty,
  EmptyHeader,
  EmptyTitle,
  EmptyDescription,
  EmptyContent,
  EmptyMedia,
}
````

## File: components/ui/field.tsx
````typescript
"use client"

import { useMemo } from "react"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"
import { Label } from "@/components/ui/label"
import { Separator } from "@/components/ui/separator"

function FieldSet({ className, ...props }: React.ComponentProps<"fieldset">) {
  return (
    <fieldset
      data-slot="field-set"
      className={cn(
        "flex flex-col gap-6",
        "has-[>[data-slot=checkbox-group]]:gap-3 has-[>[data-slot=radio-group]]:gap-3",
        className
      )}
      {...props}
    />
  )
}

function FieldLegend({
  className,
  variant = "legend",
  ...props
}: React.ComponentProps<"legend"> & { variant?: "legend" | "label" }) {
  return (
    <legend
      data-slot="field-legend"
      data-variant={variant}
      className={cn(
        "mb-3 font-medium",
        "data-[variant=legend]:text-base",
        "data-[variant=label]:text-sm",
        className
      )}
      {...props}
    />
  )
}

function FieldGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="field-group"
      className={cn(
        "group/field-group @container/field-group flex w-full flex-col gap-7 data-[slot=checkbox-group]:gap-3 [&>[data-slot=field-group]]:gap-4",
        className
      )}
      {...props}
    />
  )
}

const fieldVariants = cva(
  "group/field flex w-full gap-3 data-[invalid=true]:text-destructive",
  {
    variants: {
      orientation: {
        vertical: ["flex-col [&>*]:w-full [&>.sr-only]:w-auto"],
        horizontal: [
          "flex-row items-center",
          "[&>[data-slot=field-label]]:flex-auto",
          "has-[>[data-slot=field-content]]:items-start has-[>[data-slot=field-content]]:[&>[role=checkbox],[role=radio]]:mt-px",
        ],
        responsive: [
          "flex-col [&>*]:w-full [&>.sr-only]:w-auto @md/field-group:flex-row @md/field-group:items-center @md/field-group:[&>*]:w-auto",
          "@md/field-group:[&>[data-slot=field-label]]:flex-auto",
          "@md/field-group:has-[>[data-slot=field-content]]:items-start @md/field-group:has-[>[data-slot=field-content]]:[&>[role=checkbox],[role=radio]]:mt-px",
        ],
      },
    },
    defaultVariants: {
      orientation: "vertical",
    },
  }
)

function Field({
  className,
  orientation = "vertical",
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof fieldVariants>) {
  return (
    <div
      role="group"
      data-slot="field"
      data-orientation={orientation}
      className={cn(fieldVariants({ orientation }), className)}
      {...props}
    />
  )
}

function FieldContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="field-content"
      className={cn(
        "group/field-content flex flex-1 flex-col gap-1.5 leading-snug",
        className
      )}
      {...props}
    />
  )
}

function FieldLabel({
  className,
  ...props
}: React.ComponentProps<typeof Label>) {
  return (
    <Label
      data-slot="field-label"
      className={cn(
        "group/field-label peer/field-label flex w-fit gap-2 leading-snug group-data-[disabled=true]/field:opacity-50",
        "has-[>[data-slot=field]]:w-full has-[>[data-slot=field]]:flex-col has-[>[data-slot=field]]:rounded-md has-[>[data-slot=field]]:border [&>*]:data-[slot=field]:p-4",
        "has-data-[state=checked]:bg-primary/5 has-data-[state=checked]:border-primary dark:has-data-[state=checked]:bg-primary/10",
        className
      )}
      {...props}
    />
  )
}

function FieldTitle({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="field-label"
      className={cn(
        "flex w-fit items-center gap-2 text-sm leading-snug font-medium group-data-[disabled=true]/field:opacity-50",
        className
      )}
      {...props}
    />
  )
}

function FieldDescription({ className, ...props }: React.ComponentProps<"p">) {
  return (
    <p
      data-slot="field-description"
      className={cn(
        "text-muted-foreground text-sm leading-normal font-normal group-has-[[data-orientation=horizontal]]/field:text-balance",
        "last:mt-0 nth-last-2:-mt-1 [[data-variant=legend]+&]:-mt-1.5",
        "[&>a:hover]:text-primary [&>a]:underline [&>a]:underline-offset-4",
        className
      )}
      {...props}
    />
  )
}

function FieldSeparator({
  children,
  className,
  ...props
}: React.ComponentProps<"div"> & {
  children?: React.ReactNode
}) {
  return (
    <div
      data-slot="field-separator"
      data-content={!!children}
      className={cn(
        "relative -my-2 h-5 text-sm group-data-[variant=outline]/field-group:-mb-2",
        className
      )}
      {...props}
    >
      <Separator className="absolute inset-0 top-1/2" />
      {children && (
        <span
          className="bg-background text-muted-foreground relative mx-auto block w-fit px-2"
          data-slot="field-separator-content"
        >
          {children}
        </span>
      )}
    </div>
  )
}

function FieldError({
  className,
  children,
  errors,
  ...props
}: React.ComponentProps<"div"> & {
  errors?: Array<{ message?: string } | undefined>
}) {
  const content = useMemo(() => {
    if (children) {
      return children
    }

    if (!errors?.length) {
      return null
    }

    const uniqueErrors = [
      ...new Map(errors.map((error) => [error?.message, error])).values(),
    ]

    if (uniqueErrors?.length == 1) {
      return uniqueErrors[0]?.message
    }

    return (
      <ul className="ml-4 flex list-disc flex-col gap-1">
        {uniqueErrors.map(
          (error, index) =>
            error?.message && <li key={index}>{error.message}</li>
        )}
      </ul>
    )
  }, [children, errors])

  if (!content) {
    return null
  }

  return (
    <div
      role="alert"
      data-slot="field-error"
      className={cn("text-destructive text-sm font-normal", className)}
      {...props}
    >
      {content}
    </div>
  )
}

export {
  Field,
  FieldLabel,
  FieldDescription,
  FieldError,
  FieldGroup,
  FieldLegend,
  FieldSeparator,
  FieldSet,
  FieldContent,
  FieldTitle,
}
````

## File: components/ui/form.tsx
````typescript
"use client"

import * as React from "react"
import { Label as LabelPrimitive, Slot as SlotPrimitive } from "radix-ui"

import {
  Controller,
  FormProvider,
  useFormContext,
  useFormState,
  type ControllerProps,
  type FieldPath,
  type FieldValues,
} from "react-hook-form"

import { cn } from "@/lib/utils"
import { Label } from "@/components/ui/label"

const Form = FormProvider

type FormFieldContextValue<
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>,
> = {
  name: TName
}

const FormFieldContext = React.createContext<FormFieldContextValue>(
  {} as FormFieldContextValue
)

const FormField = <
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>,
>({
  ...props
}: ControllerProps<TFieldValues, TName>) => {
  return (
    <FormFieldContext.Provider value={{ name: props.name }}>
      <Controller {...props} />
    </FormFieldContext.Provider>
  )
}

const useFormField = () => {
  const fieldContext = React.useContext(FormFieldContext)
  const itemContext = React.useContext(FormItemContext)
  const { getFieldState } = useFormContext()
  const formState = useFormState({ name: fieldContext.name })
  const fieldState = getFieldState(fieldContext.name, formState)

  if (!fieldContext) {
    throw new Error("useFormField should be used within <FormField>")
  }

  const { id } = itemContext

  return {
    id,
    name: fieldContext.name,
    formItemId: `${id}-form-item`,
    formDescriptionId: `${id}-form-item-description`,
    formMessageId: `${id}-form-item-message`,
    ...fieldState,
  }
}

type FormItemContextValue = {
  id: string
}

const FormItemContext = React.createContext<FormItemContextValue>(
  {} as FormItemContextValue
)

function FormItem({ className, ...props }: React.ComponentProps<"div">) {
  const id = React.useId()

  return (
    <FormItemContext.Provider value={{ id }}>
      <div
        data-slot="form-item"
        className={cn("grid gap-2", className)}
        {...props}
      />
    </FormItemContext.Provider>
  )
}

function FormLabel({
  className,
  ...props
}: React.ComponentProps<typeof LabelPrimitive.Root>) {
  const { error, formItemId } = useFormField()

  return (
    <Label
      data-slot="form-label"
      data-error={!!error}
      className={cn("data-[error=true]:text-destructive", className)}
      htmlFor={formItemId}
      {...props}
    />
  )
}

function FormControl({ ...props }: React.ComponentProps<typeof SlotPrimitive.Slot>) {
  const { error, formItemId, formDescriptionId, formMessageId } = useFormField()

  return (
    <SlotPrimitive.Slot
      data-slot="form-control"
      id={formItemId}
      aria-describedby={
        !error
          ? `${formDescriptionId}`
          : `${formDescriptionId} ${formMessageId}`
      }
      aria-invalid={!!error}
      {...props}
    />
  )
}

function FormDescription({ className, ...props }: React.ComponentProps<"p">) {
  const { formDescriptionId } = useFormField()

  return (
    <p
      data-slot="form-description"
      id={formDescriptionId}
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

function FormMessage({ className, ...props }: React.ComponentProps<"p">) {
  const { error, formMessageId } = useFormField()
  const body = error ? String(error?.message ?? "") : props.children

  if (!body) {
    return null
  }

  return (
    <p
      data-slot="form-message"
      id={formMessageId}
      className={cn("text-destructive text-sm", className)}
      {...props}
    >
      {body}
    </p>
  )
}

export {
  useFormField,
  Form,
  FormItem,
  FormLabel,
  FormControl,
  FormDescription,
  FormMessage,
  FormField,
}
````

## File: components/ui/hover-card.tsx
````typescript
"use client"

import * as React from "react"
import { HoverCard as HoverCardPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function HoverCard({
  ...props
}: React.ComponentProps<typeof HoverCardPrimitive.Root>) {
  return <HoverCardPrimitive.Root data-slot="hover-card" {...props} />
}

function HoverCardTrigger({
  ...props
}: React.ComponentProps<typeof HoverCardPrimitive.Trigger>) {
  return (
    <HoverCardPrimitive.Trigger data-slot="hover-card-trigger" {...props} />
  )
}

function HoverCardContent({
  className,
  align = "center",
  sideOffset = 4,
  ...props
}: React.ComponentProps<typeof HoverCardPrimitive.Content>) {
  return (
    <HoverCardPrimitive.Portal data-slot="hover-card-portal">
      <HoverCardPrimitive.Content
        data-slot="hover-card-content"
        align={align}
        sideOffset={sideOffset}
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 w-64 origin-(--radix-hover-card-content-transform-origin) rounded-md border p-4 shadow-md outline-hidden",
          className
        )}
        {...props}
      />
    </HoverCardPrimitive.Portal>
  )
}

export { HoverCard, HoverCardTrigger, HoverCardContent }
````

## File: components/ui/input-group.tsx
````typescript
"use client"

import * as React from "react"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Textarea } from "@/components/ui/textarea"

function InputGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="input-group"
      role="group"
      className={cn(
        "group/input-group border-input dark:bg-input/30 relative flex w-full items-center rounded-md border shadow-xs transition-[color,box-shadow] outline-none",
        "h-9 min-w-0 has-[>textarea]:h-auto",

        // Variants based on alignment.
        "has-[>[data-align=inline-start]]:[&>input]:pl-2",
        "has-[>[data-align=inline-end]]:[&>input]:pr-2",
        "has-[>[data-align=block-start]]:h-auto has-[>[data-align=block-start]]:flex-col has-[>[data-align=block-start]]:[&>input]:pb-3",
        "has-[>[data-align=block-end]]:h-auto has-[>[data-align=block-end]]:flex-col has-[>[data-align=block-end]]:[&>input]:pt-3",

        // Focus state.
        "has-[[data-slot=input-group-control]:focus-visible]:border-ring has-[[data-slot=input-group-control]:focus-visible]:ring-ring/50 has-[[data-slot=input-group-control]:focus-visible]:ring-[3px]",

        // Error state.
        "has-[[data-slot][aria-invalid=true]]:ring-destructive/20 has-[[data-slot][aria-invalid=true]]:border-destructive dark:has-[[data-slot][aria-invalid=true]]:ring-destructive/40",

        className
      )}
      {...props}
    />
  )
}

const inputGroupAddonVariants = cva(
  "text-muted-foreground flex h-auto cursor-text items-center justify-center gap-2 py-1.5 text-sm font-medium select-none [&>svg:not([class*='size-'])]:size-4 [&>kbd]:rounded-[calc(var(--radius)-5px)] group-data-[disabled=true]/input-group:opacity-50",
  {
    variants: {
      align: {
        "inline-start":
          "order-first pl-3 has-[>button]:ml-[-0.45rem] has-[>kbd]:ml-[-0.35rem]",
        "inline-end":
          "order-last pr-3 has-[>button]:mr-[-0.45rem] has-[>kbd]:mr-[-0.35rem]",
        "block-start":
          "order-first w-full justify-start px-3 pt-3 [.border-b]:pb-3 group-has-[>input]/input-group:pt-2.5",
        "block-end":
          "order-last w-full justify-start px-3 pb-3 [.border-t]:pt-3 group-has-[>input]/input-group:pb-2.5",
      },
    },
    defaultVariants: {
      align: "inline-start",
    },
  }
)

function InputGroupAddon({
  className,
  align = "inline-start",
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof inputGroupAddonVariants>) {
  return (
    <div
      role="group"
      data-slot="input-group-addon"
      data-align={align}
      className={cn(inputGroupAddonVariants({ align }), className)}
      onClick={(e) => {
        if ((e.target as HTMLElement).closest("button")) {
          return
        }
        e.currentTarget.parentElement?.querySelector("input")?.focus()
      }}
      {...props}
    />
  )
}

const inputGroupButtonVariants = cva(
  "text-sm shadow-none flex gap-2 items-center",
  {
    variants: {
      size: {
        xs: "h-6 gap-1 px-2 rounded-[calc(var(--radius)-5px)] [&>svg:not([class*='size-'])]:size-3.5 has-[>svg]:px-2",
        sm: "h-8 px-2.5 gap-1.5 rounded-md has-[>svg]:px-2.5",
        "icon-xs":
          "size-6 rounded-[calc(var(--radius)-5px)] p-0 has-[>svg]:p-0",
        "icon-sm": "size-8 p-0 has-[>svg]:p-0",
      },
    },
    defaultVariants: {
      size: "xs",
    },
  }
)

function InputGroupButton({
  className,
  type = "button",
  variant = "ghost",
  size = "xs",
  ...props
}: Omit<React.ComponentProps<typeof Button>, "size"> &
  VariantProps<typeof inputGroupButtonVariants>) {
  return (
    <Button
      type={type}
      data-size={size}
      variant={variant}
      className={cn(inputGroupButtonVariants({ size }), className)}
      {...props}
    />
  )
}

function InputGroupText({ className, ...props }: React.ComponentProps<"span">) {
  return (
    <span
      className={cn(
        "text-muted-foreground flex items-center gap-2 text-sm [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function InputGroupInput({
  className,
  ...props
}: React.ComponentProps<"input">) {
  return (
    <Input
      data-slot="input-group-control"
      className={cn(
        "flex-1 rounded-none border-0 bg-transparent shadow-none focus-visible:ring-0 dark:bg-transparent",
        className
      )}
      {...props}
    />
  )
}

function InputGroupTextarea({
  className,
  ...props
}: React.ComponentProps<"textarea">) {
  return (
    <Textarea
      data-slot="input-group-control"
      className={cn(
        "flex-1 resize-none rounded-none border-0 bg-transparent py-3 shadow-none focus-visible:ring-0 dark:bg-transparent",
        className
      )}
      {...props}
    />
  )
}

export {
  InputGroup,
  InputGroupAddon,
  InputGroupButton,
  InputGroupText,
  InputGroupInput,
  InputGroupTextarea,
}
````

## File: components/ui/input-otp.tsx
````typescript
"use client"

import * as React from "react"
import { OTPInput, OTPInputContext } from "input-otp"
import { MinusIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function InputOTP({
  className,
  containerClassName,
  ...props
}: React.ComponentProps<typeof OTPInput> & {
  containerClassName?: string
}) {
  return (
    <OTPInput
      data-slot="input-otp"
      containerClassName={cn(
        "flex items-center gap-2 has-disabled:opacity-50",
        containerClassName
      )}
      className={cn("disabled:cursor-not-allowed", className)}
      {...props}
    />
  )
}

function InputOTPGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="input-otp-group"
      className={cn("flex items-center", className)}
      {...props}
    />
  )
}

function InputOTPSlot({
  index,
  className,
  ...props
}: React.ComponentProps<"div"> & {
  index: number
}) {
  const inputOTPContext = React.useContext(OTPInputContext)
  const { char, hasFakeCaret, isActive } = inputOTPContext?.slots[index] ?? {}

  return (
    <div
      data-slot="input-otp-slot"
      data-active={isActive}
      className={cn(
        "data-[active=true]:border-ring data-[active=true]:ring-ring/50 data-[active=true]:aria-invalid:ring-destructive/20 dark:data-[active=true]:aria-invalid:ring-destructive/40 aria-invalid:border-destructive data-[active=true]:aria-invalid:border-destructive dark:bg-input/30 border-input relative flex h-9 w-9 items-center justify-center border-y border-r text-sm shadow-xs transition-all outline-none first:rounded-l-md first:border-l last:rounded-r-md data-[active=true]:z-10 data-[active=true]:ring-[3px]",
        className
      )}
      {...props}
    >
      {char}
      {hasFakeCaret && (
        <div className="pointer-events-none absolute inset-0 flex items-center justify-center">
          <div className="animate-caret-blink bg-foreground h-4 w-px duration-1000" />
        </div>
      )}
    </div>
  )
}

function InputOTPSeparator({ ...props }: React.ComponentProps<"div">) {
  return (
    <div data-slot="input-otp-separator" role="separator" {...props}>
      <MinusIcon />
    </div>
  )
}

export { InputOTP, InputOTPGroup, InputOTPSlot, InputOTPSeparator }
````

## File: components/ui/input.tsx
````typescript
import * as React from "react"

import { cn } from "@/lib/utils"

function Input({ className, type, ...props }: React.ComponentProps<"input">) {
  return (
    <input
      type={type}
      data-slot="input"
      className={cn(
        "file:text-foreground placeholder:text-muted-foreground selection:bg-primary selection:text-primary-foreground dark:bg-input/30 border-input h-9 w-full min-w-0 rounded-md border bg-transparent px-3 py-1 text-base shadow-xs transition-[color,box-shadow] outline-none file:inline-flex file:h-7 file:border-0 file:bg-transparent file:text-sm file:font-medium disabled:pointer-events-none disabled:cursor-not-allowed disabled:opacity-50 md:text-sm",
        "focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px]",
        "aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive",
        className
      )}
      {...props}
    />
  )
}

export { Input }
````

## File: components/ui/item.tsx
````typescript
import * as React from "react"
import { Slot as SlotPrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"
import { Separator } from "@/components/ui/separator"

function ItemGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      role="list"
      data-slot="item-group"
      className={cn("group/item-group flex flex-col", className)}
      {...props}
    />
  )
}

function ItemSeparator({
  className,
  ...props
}: React.ComponentProps<typeof Separator>) {
  return (
    <Separator
      data-slot="item-separator"
      orientation="horizontal"
      className={cn("my-0", className)}
      {...props}
    />
  )
}

const itemVariants = cva(
  "group/item flex items-center border border-transparent text-sm rounded-md transition-colors [a]:hover:bg-accent/50 [a]:transition-colors duration-100 flex-wrap outline-none focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px]",
  {
    variants: {
      variant: {
        default: "bg-transparent",
        outline: "border-border",
        muted: "bg-muted/50",
      },
      size: {
        default: "p-4 gap-4 ",
        sm: "py-3 px-4 gap-2.5",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function Item({
  className,
  variant = "default",
  size = "default",
  asChild = false,
  ...props
}: React.ComponentProps<"div"> &
  VariantProps<typeof itemVariants> & { asChild?: boolean }) {
  const Comp = asChild ? SlotPrimitive.Slot : "div"
  return (
    <Comp
      data-slot="item"
      data-variant={variant}
      data-size={size}
      className={cn(itemVariants({ variant, size, className }))}
      {...props}
    />
  )
}

const itemMediaVariants = cva(
  "flex shrink-0 items-center justify-center gap-2 group-has-[[data-slot=item-description]]/item:self-start [&_svg]:pointer-events-none group-has-[[data-slot=item-description]]/item:translate-y-0.5",
  {
    variants: {
      variant: {
        default: "bg-transparent",
        icon: "size-8 border rounded-sm bg-muted [&_svg:not([class*='size-'])]:size-4",
        image:
          "size-10 rounded-sm overflow-hidden [&_img]:size-full [&_img]:object-cover",
      },
    },
    defaultVariants: {
      variant: "default",
    },
  }
)

function ItemMedia({
  className,
  variant = "default",
  ...props
}: React.ComponentProps<"div"> & VariantProps<typeof itemMediaVariants>) {
  return (
    <div
      data-slot="item-media"
      data-variant={variant}
      className={cn(itemMediaVariants({ variant, className }))}
      {...props}
    />
  )
}

function ItemContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="item-content"
      className={cn(
        "flex flex-1 flex-col gap-1 [&+[data-slot=item-content]]:flex-none",
        className
      )}
      {...props}
    />
  )
}

function ItemTitle({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="item-title"
      className={cn(
        "flex w-fit items-center gap-2 text-sm leading-snug font-medium",
        className
      )}
      {...props}
    />
  )
}

function ItemDescription({ className, ...props }: React.ComponentProps<"p">) {
  return (
    <p
      data-slot="item-description"
      className={cn(
        "text-muted-foreground line-clamp-2 text-sm leading-normal font-normal text-balance",
        "[&>a:hover]:text-primary [&>a]:underline [&>a]:underline-offset-4",
        className
      )}
      {...props}
    />
  )
}

function ItemActions({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="item-actions"
      className={cn("flex items-center gap-2", className)}
      {...props}
    />
  )
}

function ItemHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="item-header"
      className={cn(
        "flex basis-full items-center justify-between gap-2",
        className
      )}
      {...props}
    />
  )
}

function ItemFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="item-footer"
      className={cn(
        "flex basis-full items-center justify-between gap-2",
        className
      )}
      {...props}
    />
  )
}

export {
  Item,
  ItemMedia,
  ItemContent,
  ItemActions,
  ItemGroup,
  ItemSeparator,
  ItemTitle,
  ItemDescription,
  ItemHeader,
  ItemFooter,
}
````

## File: components/ui/kbd.tsx
````typescript
import { cn } from "@/lib/utils"

function Kbd({ className, ...props }: React.ComponentProps<"kbd">) {
  return (
    <kbd
      data-slot="kbd"
      className={cn(
        "bg-muted text-muted-foreground pointer-events-none inline-flex h-5 w-fit min-w-5 items-center justify-center gap-1 rounded-sm px-1 font-sans text-xs font-medium select-none",
        "[&_svg:not([class*='size-'])]:size-3",
        "[[data-slot=tooltip-content]_&]:bg-background/20 [[data-slot=tooltip-content]_&]:text-background dark:[[data-slot=tooltip-content]_&]:bg-background/10",
        className
      )}
      {...props}
    />
  )
}

function KbdGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <kbd
      data-slot="kbd-group"
      className={cn("inline-flex items-center gap-1", className)}
      {...props}
    />
  )
}

export { Kbd, KbdGroup }
````

## File: components/ui/label.tsx
````typescript
"use client"

import * as React from "react"
import { Label as LabelPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Label({
  className,
  ...props
}: React.ComponentProps<typeof LabelPrimitive.Root>) {
  return (
    <LabelPrimitive.Root
      data-slot="label"
      className={cn(
        "flex items-center gap-2 text-sm leading-none font-medium select-none group-data-[disabled=true]:pointer-events-none group-data-[disabled=true]:opacity-50 peer-disabled:cursor-not-allowed peer-disabled:opacity-50",
        className
      )}
      {...props}
    />
  )
}

export { Label }
````

## File: components/ui/menubar.tsx
````typescript
"use client"

import * as React from "react"
import { Menubar as MenubarPrimitive } from "radix-ui"
import { CheckIcon, ChevronRightIcon, CircleIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Menubar({
  className,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Root>) {
  return (
    <MenubarPrimitive.Root
      data-slot="menubar"
      className={cn(
        "bg-background flex h-9 items-center gap-1 rounded-md border p-1 shadow-xs",
        className
      )}
      {...props}
    />
  )
}

function MenubarMenu({
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Menu>) {
  return <MenubarPrimitive.Menu data-slot="menubar-menu" {...props} />
}

function MenubarGroup({
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Group>) {
  return <MenubarPrimitive.Group data-slot="menubar-group" {...props} />
}

function MenubarPortal({
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Portal>) {
  return <MenubarPrimitive.Portal data-slot="menubar-portal" {...props} />
}

function MenubarRadioGroup({
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.RadioGroup>) {
  return (
    <MenubarPrimitive.RadioGroup data-slot="menubar-radio-group" {...props} />
  )
}

function MenubarTrigger({
  className,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Trigger>) {
  return (
    <MenubarPrimitive.Trigger
      data-slot="menubar-trigger"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[state=open]:bg-accent data-[state=open]:text-accent-foreground flex items-center rounded-sm px-2 py-1 text-sm font-medium outline-hidden select-none",
        className
      )}
      {...props}
    />
  )
}

function MenubarContent({
  className,
  align = "start",
  alignOffset = -4,
  sideOffset = 8,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Content>) {
  return (
    <MenubarPortal>
      <MenubarPrimitive.Content
        data-slot="menubar-content"
        align={align}
        alignOffset={alignOffset}
        sideOffset={sideOffset}
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 min-w-[12rem] origin-(--radix-menubar-content-transform-origin) overflow-hidden rounded-md border p-1 shadow-md",
          className
        )}
        {...props}
      />
    </MenubarPortal>
  )
}

function MenubarItem({
  className,
  inset,
  variant = "default",
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Item> & {
  inset?: boolean
  variant?: "default" | "destructive"
}) {
  return (
    <MenubarPrimitive.Item
      data-slot="menubar-item"
      data-inset={inset}
      data-variant={variant}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[variant=destructive]:text-destructive data-[variant=destructive]:focus:bg-destructive/10 dark:data-[variant=destructive]:focus:bg-destructive/20 data-[variant=destructive]:focus:text-destructive data-[variant=destructive]:*:[svg]:!text-destructive [&_svg:not([class*='text-'])]:text-muted-foreground relative flex cursor-default items-center gap-2 rounded-sm px-2 py-1.5 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 data-[inset]:pl-8 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function MenubarCheckboxItem({
  className,
  children,
  checked,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.CheckboxItem>) {
  return (
    <MenubarPrimitive.CheckboxItem
      data-slot="menubar-checkbox-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-xs py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      checked={checked}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <MenubarPrimitive.ItemIndicator>
          <CheckIcon className="size-4" />
        </MenubarPrimitive.ItemIndicator>
      </span>
      {children}
    </MenubarPrimitive.CheckboxItem>
  )
}

function MenubarRadioItem({
  className,
  children,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.RadioItem>) {
  return (
    <MenubarPrimitive.RadioItem
      data-slot="menubar-radio-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground relative flex cursor-default items-center gap-2 rounded-xs py-1.5 pr-2 pl-8 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      <span className="pointer-events-none absolute left-2 flex size-3.5 items-center justify-center">
        <MenubarPrimitive.ItemIndicator>
          <CircleIcon className="size-2 fill-current" />
        </MenubarPrimitive.ItemIndicator>
      </span>
      {children}
    </MenubarPrimitive.RadioItem>
  )
}

function MenubarLabel({
  className,
  inset,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Label> & {
  inset?: boolean
}) {
  return (
    <MenubarPrimitive.Label
      data-slot="menubar-label"
      data-inset={inset}
      className={cn(
        "px-2 py-1.5 text-sm font-medium data-[inset]:pl-8",
        className
      )}
      {...props}
    />
  )
}

function MenubarSeparator({
  className,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Separator>) {
  return (
    <MenubarPrimitive.Separator
      data-slot="menubar-separator"
      className={cn("bg-border -mx-1 my-1 h-px", className)}
      {...props}
    />
  )
}

function MenubarShortcut({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      data-slot="menubar-shortcut"
      className={cn(
        "text-muted-foreground ml-auto text-xs tracking-widest",
        className
      )}
      {...props}
    />
  )
}

function MenubarSub({
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.Sub>) {
  return <MenubarPrimitive.Sub data-slot="menubar-sub" {...props} />
}

function MenubarSubTrigger({
  className,
  inset,
  children,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.SubTrigger> & {
  inset?: boolean
}) {
  return (
    <MenubarPrimitive.SubTrigger
      data-slot="menubar-sub-trigger"
      data-inset={inset}
      className={cn(
        "focus:bg-accent focus:text-accent-foreground data-[state=open]:bg-accent data-[state=open]:text-accent-foreground flex cursor-default items-center rounded-sm px-2 py-1.5 text-sm outline-none select-none data-[inset]:pl-8",
        className
      )}
      {...props}
    >
      {children}
      <ChevronRightIcon className="ml-auto h-4 w-4" />
    </MenubarPrimitive.SubTrigger>
  )
}

function MenubarSubContent({
  className,
  ...props
}: React.ComponentProps<typeof MenubarPrimitive.SubContent>) {
  return (
    <MenubarPrimitive.SubContent
      data-slot="menubar-sub-content"
      className={cn(
        "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 min-w-[8rem] origin-(--radix-menubar-content-transform-origin) overflow-hidden rounded-md border p-1 shadow-lg",
        className
      )}
      {...props}
    />
  )
}

export {
  Menubar,
  MenubarPortal,
  MenubarMenu,
  MenubarTrigger,
  MenubarContent,
  MenubarGroup,
  MenubarSeparator,
  MenubarLabel,
  MenubarItem,
  MenubarShortcut,
  MenubarCheckboxItem,
  MenubarRadioGroup,
  MenubarRadioItem,
  MenubarSub,
  MenubarSubTrigger,
  MenubarSubContent,
}
````

## File: components/ui/navigation-menu.tsx
````typescript
import * as React from "react"
import { NavigationMenu as NavigationMenuPrimitive } from "radix-ui"
import { cva } from "class-variance-authority"
import { ChevronDownIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function NavigationMenu({
  className,
  children,
  viewport = true,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Root> & {
  viewport?: boolean
}) {
  return (
    <NavigationMenuPrimitive.Root
      data-slot="navigation-menu"
      data-viewport={viewport}
      className={cn(
        "group/navigation-menu relative flex max-w-max flex-1 items-center justify-center",
        className
      )}
      {...props}
    >
      {children}
      {viewport && <NavigationMenuViewport />}
    </NavigationMenuPrimitive.Root>
  )
}

function NavigationMenuList({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.List>) {
  return (
    <NavigationMenuPrimitive.List
      data-slot="navigation-menu-list"
      className={cn(
        "group flex flex-1 list-none items-center justify-center gap-1",
        className
      )}
      {...props}
    />
  )
}

function NavigationMenuItem({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Item>) {
  return (
    <NavigationMenuPrimitive.Item
      data-slot="navigation-menu-item"
      className={cn("relative", className)}
      {...props}
    />
  )
}

const navigationMenuTriggerStyle = cva(
  "group inline-flex h-9 w-max items-center justify-center rounded-md bg-background px-4 py-2 text-sm font-medium hover:bg-accent hover:text-accent-foreground focus:bg-accent focus:text-accent-foreground disabled:pointer-events-none disabled:opacity-50 data-[state=open]:hover:bg-accent data-[state=open]:text-accent-foreground data-[state=open]:focus:bg-accent data-[state=open]:bg-accent/50 focus-visible:ring-ring/50 outline-none transition-[color,box-shadow] focus-visible:ring-[3px] focus-visible:outline-1"
)

function NavigationMenuTrigger({
  className,
  children,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Trigger>) {
  return (
    <NavigationMenuPrimitive.Trigger
      data-slot="navigation-menu-trigger"
      className={cn(navigationMenuTriggerStyle(), "group", className)}
      {...props}
    >
      {children}{" "}
      <ChevronDownIcon
        className="relative top-[1px] ml-1 size-3 transition duration-300 group-data-[state=open]:rotate-180"
        aria-hidden="true"
      />
    </NavigationMenuPrimitive.Trigger>
  )
}

function NavigationMenuContent({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Content>) {
  return (
    <NavigationMenuPrimitive.Content
      data-slot="navigation-menu-content"
      className={cn(
        "data-[motion^=from-]:animate-in data-[motion^=to-]:animate-out data-[motion^=from-]:fade-in data-[motion^=to-]:fade-out data-[motion=from-end]:slide-in-from-right-52 data-[motion=from-start]:slide-in-from-left-52 data-[motion=to-end]:slide-out-to-right-52 data-[motion=to-start]:slide-out-to-left-52 top-0 left-0 w-full p-2 pr-2.5 md:absolute md:w-auto",
        "group-data-[viewport=false]/navigation-menu:bg-popover group-data-[viewport=false]/navigation-menu:text-popover-foreground group-data-[viewport=false]/navigation-menu:data-[state=open]:animate-in group-data-[viewport=false]/navigation-menu:data-[state=closed]:animate-out group-data-[viewport=false]/navigation-menu:data-[state=closed]:zoom-out-95 group-data-[viewport=false]/navigation-menu:data-[state=open]:zoom-in-95 group-data-[viewport=false]/navigation-menu:data-[state=open]:fade-in-0 group-data-[viewport=false]/navigation-menu:data-[state=closed]:fade-out-0 group-data-[viewport=false]/navigation-menu:top-full group-data-[viewport=false]/navigation-menu:mt-1.5 group-data-[viewport=false]/navigation-menu:overflow-hidden group-data-[viewport=false]/navigation-menu:rounded-md group-data-[viewport=false]/navigation-menu:border group-data-[viewport=false]/navigation-menu:shadow group-data-[viewport=false]/navigation-menu:duration-200 **:data-[slot=navigation-menu-link]:focus:ring-0 **:data-[slot=navigation-menu-link]:focus:outline-none",
        className
      )}
      {...props}
    />
  )
}

function NavigationMenuViewport({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Viewport>) {
  return (
    <div
      className={cn(
        "absolute top-full left-0 isolate z-50 flex justify-center"
      )}
    >
      <NavigationMenuPrimitive.Viewport
        data-slot="navigation-menu-viewport"
        className={cn(
          "origin-top-center bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-90 relative mt-1.5 h-[var(--radix-navigation-menu-viewport-height)] w-full overflow-hidden rounded-md border shadow md:w-[var(--radix-navigation-menu-viewport-width)]",
          className
        )}
        {...props}
      />
    </div>
  )
}

function NavigationMenuLink({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Link>) {
  return (
    <NavigationMenuPrimitive.Link
      data-slot="navigation-menu-link"
      className={cn(
        "data-[active=true]:focus:bg-accent data-[active=true]:hover:bg-accent data-[active=true]:bg-accent/50 data-[active=true]:text-accent-foreground hover:bg-accent hover:text-accent-foreground focus:bg-accent focus:text-accent-foreground focus-visible:ring-ring/50 [&_svg:not([class*='text-'])]:text-muted-foreground flex flex-col gap-1 rounded-sm p-2 text-sm transition-all outline-none focus-visible:ring-[3px] focus-visible:outline-1 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function NavigationMenuIndicator({
  className,
  ...props
}: React.ComponentProps<typeof NavigationMenuPrimitive.Indicator>) {
  return (
    <NavigationMenuPrimitive.Indicator
      data-slot="navigation-menu-indicator"
      className={cn(
        "data-[state=visible]:animate-in data-[state=hidden]:animate-out data-[state=hidden]:fade-out data-[state=visible]:fade-in top-full z-[1] flex h-1.5 items-end justify-center overflow-hidden",
        className
      )}
      {...props}
    >
      <div className="bg-border relative top-[60%] h-2 w-2 rotate-45 rounded-tl-sm shadow-md" />
    </NavigationMenuPrimitive.Indicator>
  )
}

export {
  NavigationMenu,
  NavigationMenuList,
  NavigationMenuItem,
  NavigationMenuContent,
  NavigationMenuTrigger,
  NavigationMenuLink,
  NavigationMenuIndicator,
  NavigationMenuViewport,
  navigationMenuTriggerStyle,
}
````

## File: components/ui/pagination.tsx
````typescript
import * as React from "react"
import {
  ChevronLeftIcon,
  ChevronRightIcon,
  MoreHorizontalIcon,
} from "lucide-react"

import { cn } from "@/lib/utils"
import { Button, buttonVariants } from "@/components/ui/button"

function Pagination({ className, ...props }: React.ComponentProps<"nav">) {
  return (
    <nav
      role="navigation"
      aria-label="pagination"
      data-slot="pagination"
      className={cn("mx-auto flex w-full justify-center", className)}
      {...props}
    />
  )
}

function PaginationContent({
  className,
  ...props
}: React.ComponentProps<"ul">) {
  return (
    <ul
      data-slot="pagination-content"
      className={cn("flex flex-row items-center gap-1", className)}
      {...props}
    />
  )
}

function PaginationItem({ ...props }: React.ComponentProps<"li">) {
  return <li data-slot="pagination-item" {...props} />
}

type PaginationLinkProps = {
  isActive?: boolean
} & Pick<React.ComponentProps<typeof Button>, "size"> &
  React.ComponentProps<"a">

function PaginationLink({
  className,
  isActive,
  size = "icon",
  ...props
}: PaginationLinkProps) {
  return (
    <a
      aria-current={isActive ? "page" : undefined}
      data-slot="pagination-link"
      data-active={isActive}
      className={cn(
        buttonVariants({
          variant: isActive ? "outline" : "ghost",
          size,
        }),
        className
      )}
      {...props}
    />
  )
}

function PaginationPrevious({
  className,
  ...props
}: React.ComponentProps<typeof PaginationLink>) {
  return (
    <PaginationLink
      aria-label="Go to previous page"
      size="default"
      className={cn("gap-1 px-2.5 sm:pl-2.5", className)}
      {...props}
    >
      <ChevronLeftIcon />
      <span className="hidden sm:block">Previous</span>
    </PaginationLink>
  )
}

function PaginationNext({
  className,
  ...props
}: React.ComponentProps<typeof PaginationLink>) {
  return (
    <PaginationLink
      aria-label="Go to next page"
      size="default"
      className={cn("gap-1 px-2.5 sm:pr-2.5", className)}
      {...props}
    >
      <span className="hidden sm:block">Next</span>
      <ChevronRightIcon />
    </PaginationLink>
  )
}

function PaginationEllipsis({
  className,
  ...props
}: React.ComponentProps<"span">) {
  return (
    <span
      aria-hidden
      data-slot="pagination-ellipsis"
      className={cn("flex size-9 items-center justify-center", className)}
      {...props}
    >
      <MoreHorizontalIcon className="size-4" />
      <span className="sr-only">More pages</span>
    </span>
  )
}

export {
  Pagination,
  PaginationContent,
  PaginationLink,
  PaginationItem,
  PaginationPrevious,
  PaginationNext,
  PaginationEllipsis,
}
````

## File: components/ui/popover.tsx
````typescript
"use client"

import * as React from "react"
import { Popover as PopoverPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Popover({
  ...props
}: React.ComponentProps<typeof PopoverPrimitive.Root>) {
  return <PopoverPrimitive.Root data-slot="popover" {...props} />
}

function PopoverTrigger({
  ...props
}: React.ComponentProps<typeof PopoverPrimitive.Trigger>) {
  return <PopoverPrimitive.Trigger data-slot="popover-trigger" {...props} />
}

function PopoverContent({
  className,
  align = "center",
  sideOffset = 4,
  ...props
}: React.ComponentProps<typeof PopoverPrimitive.Content>) {
  return (
    <PopoverPrimitive.Portal>
      <PopoverPrimitive.Content
        data-slot="popover-content"
        align={align}
        sideOffset={sideOffset}
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 w-72 origin-(--radix-popover-content-transform-origin) rounded-md border p-4 shadow-md outline-hidden",
          className
        )}
        {...props}
      />
    </PopoverPrimitive.Portal>
  )
}

function PopoverAnchor({
  ...props
}: React.ComponentProps<typeof PopoverPrimitive.Anchor>) {
  return <PopoverPrimitive.Anchor data-slot="popover-anchor" {...props} />
}

export { Popover, PopoverTrigger, PopoverContent, PopoverAnchor }
````

## File: components/ui/progress.tsx
````typescript
"use client"

import * as React from "react"
import { Progress as ProgressPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Progress({
  className,
  value,
  ...props
}: React.ComponentProps<typeof ProgressPrimitive.Root>) {
  return (
    <ProgressPrimitive.Root
      data-slot="progress"
      className={cn(
        "bg-primary/20 relative h-2 w-full overflow-hidden rounded-full",
        className
      )}
      {...props}
    >
      <ProgressPrimitive.Indicator
        data-slot="progress-indicator"
        className="bg-primary h-full w-full flex-1 transition-all"
        style={{ transform: `translateX(-${100 - (value || 0)}%)` }}
      />
    </ProgressPrimitive.Root>
  )
}

export { Progress }
````

## File: components/ui/radio-group.tsx
````typescript
"use client"

import * as React from "react"
import { RadioGroup as RadioGroupPrimitive } from "radix-ui"
import { CircleIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function RadioGroup({
  className,
  ...props
}: React.ComponentProps<typeof RadioGroupPrimitive.Root>) {
  return (
    <RadioGroupPrimitive.Root
      data-slot="radio-group"
      className={cn("grid gap-3", className)}
      {...props}
    />
  )
}

function RadioGroupItem({
  className,
  ...props
}: React.ComponentProps<typeof RadioGroupPrimitive.Item>) {
  return (
    <RadioGroupPrimitive.Item
      data-slot="radio-group-item"
      className={cn(
        "border-input text-primary focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive dark:bg-input/30 aspect-square size-4 shrink-0 rounded-full border shadow-xs transition-[color,box-shadow] outline-none focus-visible:ring-[3px] disabled:cursor-not-allowed disabled:opacity-50",
        className
      )}
      {...props}
    >
      <RadioGroupPrimitive.Indicator
        data-slot="radio-group-indicator"
        className="relative flex items-center justify-center"
      >
        <CircleIcon className="fill-primary absolute top-1/2 left-1/2 size-2 -translate-x-1/2 -translate-y-1/2" />
      </RadioGroupPrimitive.Indicator>
    </RadioGroupPrimitive.Item>
  )
}

export { RadioGroup, RadioGroupItem }
````

## File: components/ui/resizable.tsx
````typescript
"use client"

import * as React from "react"
import { GripVerticalIcon } from "lucide-react"
import * as ResizablePrimitive from "react-resizable-panels"

import { cn } from "@/lib/utils"

function ResizablePanelGroup({
  className,
  ...props
}: React.ComponentProps<typeof ResizablePrimitive.PanelGroup>) {
  return (
    <ResizablePrimitive.PanelGroup
      data-slot="resizable-panel-group"
      className={cn(
        "flex h-full w-full data-[panel-group-direction=vertical]:flex-col",
        className
      )}
      {...props}
    />
  )
}

function ResizablePanel({
  ...props
}: React.ComponentProps<typeof ResizablePrimitive.Panel>) {
  return <ResizablePrimitive.Panel data-slot="resizable-panel" {...props} />
}

function ResizableHandle({
  withHandle,
  className,
  ...props
}: React.ComponentProps<typeof ResizablePrimitive.PanelResizeHandle> & {
  withHandle?: boolean
}) {
  return (
    <ResizablePrimitive.PanelResizeHandle
      data-slot="resizable-handle"
      className={cn(
        "bg-border focus-visible:ring-ring relative flex w-px items-center justify-center after:absolute after:inset-y-0 after:left-1/2 after:w-1 after:-translate-x-1/2 focus-visible:ring-1 focus-visible:ring-offset-1 focus-visible:outline-hidden data-[panel-group-direction=vertical]:h-px data-[panel-group-direction=vertical]:w-full data-[panel-group-direction=vertical]:after:left-0 data-[panel-group-direction=vertical]:after:h-1 data-[panel-group-direction=vertical]:after:w-full data-[panel-group-direction=vertical]:after:translate-x-0 data-[panel-group-direction=vertical]:after:-translate-y-1/2 [&[data-panel-group-direction=vertical]>div]:rotate-90",
        className
      )}
      {...props}
    >
      {withHandle && (
        <div className="bg-border z-10 flex h-4 w-3 items-center justify-center rounded-xs border">
          <GripVerticalIcon className="size-2.5" />
        </div>
      )}
    </ResizablePrimitive.PanelResizeHandle>
  )
}

export { ResizablePanelGroup, ResizablePanel, ResizableHandle }
````

## File: components/ui/scroll-area.tsx
````typescript
"use client"

import * as React from "react"
import { ScrollArea as ScrollAreaPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function ScrollArea({
  className,
  children,
  ...props
}: React.ComponentProps<typeof ScrollAreaPrimitive.Root>) {
  return (
    <ScrollAreaPrimitive.Root
      data-slot="scroll-area"
      className={cn("relative", className)}
      {...props}
    >
      <ScrollAreaPrimitive.Viewport
        data-slot="scroll-area-viewport"
        className="focus-visible:ring-ring/50 size-full rounded-[inherit] transition-[color,box-shadow] outline-none focus-visible:ring-[3px] focus-visible:outline-1"
      >
        {children}
      </ScrollAreaPrimitive.Viewport>
      <ScrollBar />
      <ScrollAreaPrimitive.Corner />
    </ScrollAreaPrimitive.Root>
  )
}

function ScrollBar({
  className,
  orientation = "vertical",
  ...props
}: React.ComponentProps<typeof ScrollAreaPrimitive.ScrollAreaScrollbar>) {
  return (
    <ScrollAreaPrimitive.ScrollAreaScrollbar
      data-slot="scroll-area-scrollbar"
      orientation={orientation}
      className={cn(
        "flex touch-none p-px transition-colors select-none",
        orientation === "vertical" &&
          "h-full w-2.5 border-l border-l-transparent",
        orientation === "horizontal" &&
          "h-2.5 flex-col border-t border-t-transparent",
        className
      )}
      {...props}
    >
      <ScrollAreaPrimitive.ScrollAreaThumb
        data-slot="scroll-area-thumb"
        className="bg-border relative flex-1 rounded-full"
      />
    </ScrollAreaPrimitive.ScrollAreaScrollbar>
  )
}

export { ScrollArea, ScrollBar }
````

## File: components/ui/select.tsx
````typescript
"use client"

import * as React from "react"
import { Select as SelectPrimitive } from "radix-ui"
import { CheckIcon, ChevronDownIcon, ChevronUpIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Select({
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Root>) {
  return <SelectPrimitive.Root data-slot="select" {...props} />
}

function SelectGroup({
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Group>) {
  return <SelectPrimitive.Group data-slot="select-group" {...props} />
}

function SelectValue({
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Value>) {
  return <SelectPrimitive.Value data-slot="select-value" {...props} />
}

function SelectTrigger({
  className,
  size = "default",
  children,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Trigger> & {
  size?: "sm" | "default"
}) {
  return (
    <SelectPrimitive.Trigger
      data-slot="select-trigger"
      data-size={size}
      className={cn(
        "border-input data-[placeholder]:text-muted-foreground [&_svg:not([class*='text-'])]:text-muted-foreground focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive dark:bg-input/30 dark:hover:bg-input/50 flex w-fit items-center justify-between gap-2 rounded-md border bg-transparent px-3 py-2 text-sm whitespace-nowrap shadow-xs transition-[color,box-shadow] outline-none focus-visible:ring-[3px] disabled:cursor-not-allowed disabled:opacity-50 data-[size=default]:h-9 data-[size=sm]:h-8 *:data-[slot=select-value]:line-clamp-1 *:data-[slot=select-value]:flex *:data-[slot=select-value]:items-center *:data-[slot=select-value]:gap-2 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    >
      {children}
      <SelectPrimitive.Icon asChild>
        <ChevronDownIcon className="size-4 opacity-50" />
      </SelectPrimitive.Icon>
    </SelectPrimitive.Trigger>
  )
}

function SelectContent({
  className,
  children,
  position = "popper",
  align = "center",
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Content>) {
  return (
    <SelectPrimitive.Portal>
      <SelectPrimitive.Content
        data-slot="select-content"
        className={cn(
          "bg-popover text-popover-foreground data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 relative z-50 max-h-(--radix-select-content-available-height) min-w-[8rem] origin-(--radix-select-content-transform-origin) overflow-x-hidden overflow-y-auto rounded-md border shadow-md",
          position === "popper" &&
            "data-[side=bottom]:translate-y-1 data-[side=left]:-translate-x-1 data-[side=right]:translate-x-1 data-[side=top]:-translate-y-1",
          className
        )}
        position={position}
        align={align}
        {...props}
      >
        <SelectScrollUpButton />
        <SelectPrimitive.Viewport
          className={cn(
            "p-1",
            position === "popper" &&
              "h-[var(--radix-select-trigger-height)] w-full min-w-[var(--radix-select-trigger-width)] scroll-my-1"
          )}
        >
          {children}
        </SelectPrimitive.Viewport>
        <SelectScrollDownButton />
      </SelectPrimitive.Content>
    </SelectPrimitive.Portal>
  )
}

function SelectLabel({
  className,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Label>) {
  return (
    <SelectPrimitive.Label
      data-slot="select-label"
      className={cn("text-muted-foreground px-2 py-1.5 text-xs", className)}
      {...props}
    />
  )
}

function SelectItem({
  className,
  children,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Item>) {
  return (
    <SelectPrimitive.Item
      data-slot="select-item"
      className={cn(
        "focus:bg-accent focus:text-accent-foreground [&_svg:not([class*='text-'])]:text-muted-foreground relative flex w-full cursor-default items-center gap-2 rounded-sm py-1.5 pr-8 pl-2 text-sm outline-hidden select-none data-[disabled]:pointer-events-none data-[disabled]:opacity-50 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4 *:[span]:last:flex *:[span]:last:items-center *:[span]:last:gap-2",
        className
      )}
      {...props}
    >
      <span className="absolute right-2 flex size-3.5 items-center justify-center">
        <SelectPrimitive.ItemIndicator>
          <CheckIcon className="size-4" />
        </SelectPrimitive.ItemIndicator>
      </span>
      <SelectPrimitive.ItemText>{children}</SelectPrimitive.ItemText>
    </SelectPrimitive.Item>
  )
}

function SelectSeparator({
  className,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.Separator>) {
  return (
    <SelectPrimitive.Separator
      data-slot="select-separator"
      className={cn("bg-border pointer-events-none -mx-1 my-1 h-px", className)}
      {...props}
    />
  )
}

function SelectScrollUpButton({
  className,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.ScrollUpButton>) {
  return (
    <SelectPrimitive.ScrollUpButton
      data-slot="select-scroll-up-button"
      className={cn(
        "flex cursor-default items-center justify-center py-1",
        className
      )}
      {...props}
    >
      <ChevronUpIcon className="size-4" />
    </SelectPrimitive.ScrollUpButton>
  )
}

function SelectScrollDownButton({
  className,
  ...props
}: React.ComponentProps<typeof SelectPrimitive.ScrollDownButton>) {
  return (
    <SelectPrimitive.ScrollDownButton
      data-slot="select-scroll-down-button"
      className={cn(
        "flex cursor-default items-center justify-center py-1",
        className
      )}
      {...props}
    >
      <ChevronDownIcon className="size-4" />
    </SelectPrimitive.ScrollDownButton>
  )
}

export {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectLabel,
  SelectScrollDownButton,
  SelectScrollUpButton,
  SelectSeparator,
  SelectTrigger,
  SelectValue,
}
````

## File: components/ui/separator.tsx
````typescript
"use client"

import * as React from "react"
import { Separator as SeparatorPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Separator({
  className,
  orientation = "horizontal",
  decorative = true,
  ...props
}: React.ComponentProps<typeof SeparatorPrimitive.Root>) {
  return (
    <SeparatorPrimitive.Root
      data-slot="separator"
      decorative={decorative}
      orientation={orientation}
      className={cn(
        "bg-border shrink-0 data-[orientation=horizontal]:h-px data-[orientation=horizontal]:w-full data-[orientation=vertical]:h-full data-[orientation=vertical]:w-px",
        className
      )}
      {...props}
    />
  )
}

export { Separator }
````

## File: components/ui/sheet.tsx
````typescript
"use client"

import * as React from "react"
import { Dialog as SheetPrimitive } from "radix-ui"
import { XIcon } from "lucide-react"

import { cn } from "@/lib/utils"

function Sheet({ ...props }: React.ComponentProps<typeof SheetPrimitive.Root>) {
  return <SheetPrimitive.Root data-slot="sheet" {...props} />
}

function SheetTrigger({
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Trigger>) {
  return <SheetPrimitive.Trigger data-slot="sheet-trigger" {...props} />
}

function SheetClose({
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Close>) {
  return <SheetPrimitive.Close data-slot="sheet-close" {...props} />
}

function SheetPortal({
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Portal>) {
  return <SheetPrimitive.Portal data-slot="sheet-portal" {...props} />
}

function SheetOverlay({
  className,
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Overlay>) {
  return (
    <SheetPrimitive.Overlay
      data-slot="sheet-overlay"
      className={cn(
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50",
        className
      )}
      {...props}
    />
  )
}

function SheetContent({
  className,
  children,
  side = "right",
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Content> & {
  side?: "top" | "right" | "bottom" | "left"
}) {
  return (
    <SheetPortal>
      <SheetOverlay />
      <SheetPrimitive.Content
        data-slot="sheet-content"
        className={cn(
          "bg-background data-[state=open]:animate-in data-[state=closed]:animate-out fixed z-50 flex flex-col gap-4 shadow-lg transition ease-in-out data-[state=closed]:duration-300 data-[state=open]:duration-500",
          side === "right" &&
            "data-[state=closed]:slide-out-to-right data-[state=open]:slide-in-from-right inset-y-0 right-0 h-full w-3/4 border-l sm:max-w-sm",
          side === "left" &&
            "data-[state=closed]:slide-out-to-left data-[state=open]:slide-in-from-left inset-y-0 left-0 h-full w-3/4 border-r sm:max-w-sm",
          side === "top" &&
            "data-[state=closed]:slide-out-to-top data-[state=open]:slide-in-from-top inset-x-0 top-0 h-auto border-b",
          side === "bottom" &&
            "data-[state=closed]:slide-out-to-bottom data-[state=open]:slide-in-from-bottom inset-x-0 bottom-0 h-auto border-t",
          className
        )}
        {...props}
      >
        {children}
        <SheetPrimitive.Close className="ring-offset-background focus:ring-ring data-[state=open]:bg-secondary absolute top-4 right-4 rounded-xs opacity-70 transition-opacity hover:opacity-100 focus:ring-2 focus:ring-offset-2 focus:outline-hidden disabled:pointer-events-none">
          <XIcon className="size-4" />
          <span className="sr-only">Close</span>
        </SheetPrimitive.Close>
      </SheetPrimitive.Content>
    </SheetPortal>
  )
}

function SheetHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sheet-header"
      className={cn("flex flex-col gap-1.5 p-4", className)}
      {...props}
    />
  )
}

function SheetFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sheet-footer"
      className={cn("mt-auto flex flex-col gap-2 p-4", className)}
      {...props}
    />
  )
}

function SheetTitle({
  className,
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Title>) {
  return (
    <SheetPrimitive.Title
      data-slot="sheet-title"
      className={cn("text-foreground font-semibold", className)}
      {...props}
    />
  )
}

function SheetDescription({
  className,
  ...props
}: React.ComponentProps<typeof SheetPrimitive.Description>) {
  return (
    <SheetPrimitive.Description
      data-slot="sheet-description"
      className={cn("text-muted-foreground text-sm", className)}
      {...props}
    />
  )
}

export {
  Sheet,
  SheetTrigger,
  SheetClose,
  SheetContent,
  SheetHeader,
  SheetFooter,
  SheetTitle,
  SheetDescription,
}
````

## File: components/ui/sidebar.tsx
````typescript
"use client"

import * as React from "react"
import { Slot as SlotPrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"
import { PanelLeftIcon } from "lucide-react"

import { useIsMobile } from "@/hooks/use-mobile"
import { cn } from "@/lib/utils"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Separator } from "@/components/ui/separator"
import {
  Sheet,
  SheetContent,
  SheetDescription,
  SheetHeader,
  SheetTitle,
} from "@/components/ui/sheet"
import { Skeleton } from "@/components/ui/skeleton"
import {
  Tooltip,
  TooltipContent,
  TooltipProvider,
  TooltipTrigger,
} from "@/components/ui/tooltip"

const SIDEBAR_COOKIE_NAME = "sidebar_state"
const SIDEBAR_COOKIE_MAX_AGE = 60 * 60 * 24 * 7
const SIDEBAR_WIDTH = "16rem"
const SIDEBAR_WIDTH_MOBILE = "18rem"
const SIDEBAR_WIDTH_ICON = "3rem"
const SIDEBAR_KEYBOARD_SHORTCUT = "b"

type SidebarContextProps = {
  state: "expanded" | "collapsed"
  open: boolean
  setOpen: (open: boolean) => void
  openMobile: boolean
  setOpenMobile: (open: boolean) => void
  isMobile: boolean
  toggleSidebar: () => void
}

const SidebarContext = React.createContext<SidebarContextProps | null>(null)

function useSidebar() {
  const context = React.useContext(SidebarContext)
  if (!context) {
    throw new Error("useSidebar must be used within a SidebarProvider.")
  }

  return context
}

function SidebarProvider({
  defaultOpen = true,
  open: openProp,
  onOpenChange: setOpenProp,
  className,
  style,
  children,
  ...props
}: React.ComponentProps<"div"> & {
  defaultOpen?: boolean
  open?: boolean
  onOpenChange?: (open: boolean) => void
}) {
  const isMobile = useIsMobile()
  const [openMobile, setOpenMobile] = React.useState(false)

  // This is the internal state of the sidebar.
  // We use openProp and setOpenProp for control from outside the component.
  const [_open, _setOpen] = React.useState(defaultOpen)
  const open = openProp ?? _open
  const setOpen = React.useCallback(
    (value: boolean | ((value: boolean) => boolean)) => {
      const openState = typeof value === "function" ? value(open) : value
      if (setOpenProp) {
        setOpenProp(openState)
      } else {
        _setOpen(openState)
      }

      // This sets the cookie to keep the sidebar state.
      document.cookie = `${SIDEBAR_COOKIE_NAME}=${openState}; path=/; max-age=${SIDEBAR_COOKIE_MAX_AGE}`
    },
    [setOpenProp, open]
  )

  // Helper to toggle the sidebar.
  const toggleSidebar = React.useCallback(() => {
    return isMobile ? setOpenMobile((open) => !open) : setOpen((open) => !open)
  }, [isMobile, setOpen, setOpenMobile])

  // Adds a keyboard shortcut to toggle the sidebar.
  React.useEffect(() => {
    const handleKeyDown = (event: KeyboardEvent) => {
      if (
        event.key === SIDEBAR_KEYBOARD_SHORTCUT &&
        (event.metaKey || event.ctrlKey)
      ) {
        event.preventDefault()
        toggleSidebar()
      }
    }

    window.addEventListener("keydown", handleKeyDown)
    return () => window.removeEventListener("keydown", handleKeyDown)
  }, [toggleSidebar])

  // We add a state so that we can do data-state="expanded" or "collapsed".
  // This makes it easier to style the sidebar with Tailwind classes.
  const state = open ? "expanded" : "collapsed"

  const contextValue = React.useMemo<SidebarContextProps>(
    () => ({
      state,
      open,
      setOpen,
      isMobile,
      openMobile,
      setOpenMobile,
      toggleSidebar,
    }),
    [state, open, setOpen, isMobile, openMobile, setOpenMobile, toggleSidebar]
  )

  return (
    <SidebarContext.Provider value={contextValue}>
      <TooltipProvider delayDuration={0}>
        <div
          data-slot="sidebar-wrapper"
          style={
            {
              "--sidebar-width": SIDEBAR_WIDTH,
              "--sidebar-width-icon": SIDEBAR_WIDTH_ICON,
              ...style,
            } as React.CSSProperties
          }
          className={cn(
            "group/sidebar-wrapper has-data-[variant=inset]:bg-sidebar flex min-h-svh w-full",
            className
          )}
          {...props}
        >
          {children}
        </div>
      </TooltipProvider>
    </SidebarContext.Provider>
  )
}

function Sidebar({
  side = "left",
  variant = "sidebar",
  collapsible = "offcanvas",
  className,
  children,
  ...props
}: React.ComponentProps<"div"> & {
  side?: "left" | "right"
  variant?: "sidebar" | "floating" | "inset"
  collapsible?: "offcanvas" | "icon" | "none"
}) {
  const { isMobile, state, openMobile, setOpenMobile } = useSidebar()

  if (collapsible === "none") {
    return (
      <div
        data-slot="sidebar"
        className={cn(
          "bg-sidebar text-sidebar-foreground flex h-full w-(--sidebar-width) flex-col",
          className
        )}
        {...props}
      >
        {children}
      </div>
    )
  }

  if (isMobile) {
    return (
      <Sheet open={openMobile} onOpenChange={setOpenMobile} {...props}>
        <SheetContent
          data-sidebar="sidebar"
          data-slot="sidebar"
          data-mobile="true"
          className="bg-sidebar text-sidebar-foreground w-(--sidebar-width) p-0 [&>button]:hidden"
          style={
            {
              "--sidebar-width": SIDEBAR_WIDTH_MOBILE,
            } as React.CSSProperties
          }
          side={side}
        >
          <SheetHeader className="sr-only">
            <SheetTitle>Sidebar</SheetTitle>
            <SheetDescription>Displays the mobile sidebar.</SheetDescription>
          </SheetHeader>
          <div className="flex h-full w-full flex-col">{children}</div>
        </SheetContent>
      </Sheet>
    )
  }

  return (
    <div
      className="group peer text-sidebar-foreground hidden md:block"
      data-state={state}
      data-collapsible={state === "collapsed" ? collapsible : ""}
      data-variant={variant}
      data-side={side}
      data-slot="sidebar"
    >
      {/* This is what handles the sidebar gap on desktop */}
      <div
        data-slot="sidebar-gap"
        className={cn(
          "relative w-(--sidebar-width) bg-transparent transition-[width] duration-200 ease-linear",
          "group-data-[collapsible=offcanvas]:w-0",
          "group-data-[side=right]:rotate-180",
          variant === "floating" || variant === "inset"
            ? "group-data-[collapsible=icon]:w-[calc(var(--sidebar-width-icon)+(--spacing(4)))]"
            : "group-data-[collapsible=icon]:w-(--sidebar-width-icon)"
        )}
      />
      <div
        data-slot="sidebar-container"
        className={cn(
          "fixed inset-y-0 z-10 hidden h-svh w-(--sidebar-width) transition-[left,right,width] duration-200 ease-linear md:flex",
          side === "left"
            ? "left-0 group-data-[collapsible=offcanvas]:left-[calc(var(--sidebar-width)*-1)]"
            : "right-0 group-data-[collapsible=offcanvas]:right-[calc(var(--sidebar-width)*-1)]",
          // Adjust the padding for floating and inset variants.
          variant === "floating" || variant === "inset"
            ? "p-2 group-data-[collapsible=icon]:w-[calc(var(--sidebar-width-icon)+(--spacing(4))+2px)]"
            : "group-data-[collapsible=icon]:w-(--sidebar-width-icon) group-data-[side=left]:border-r group-data-[side=right]:border-l",
          className
        )}
        {...props}
      >
        <div
          data-sidebar="sidebar"
          data-slot="sidebar-inner"
          className="bg-sidebar group-data-[variant=floating]:border-sidebar-border flex h-full w-full flex-col group-data-[variant=floating]:rounded-lg group-data-[variant=floating]:border group-data-[variant=floating]:shadow-sm"
        >
          {children}
        </div>
      </div>
    </div>
  )
}

function SidebarTrigger({
  className,
  onClick,
  ...props
}: React.ComponentProps<typeof Button>) {
  const { toggleSidebar } = useSidebar()

  return (
    <Button
      data-sidebar="trigger"
      data-slot="sidebar-trigger"
      variant="ghost"
      size="icon"
      className={cn("size-7", className)}
      onClick={(event) => {
        onClick?.(event)
        toggleSidebar()
      }}
      {...props}
    >
      <PanelLeftIcon />
      <span className="sr-only">Toggle Sidebar</span>
    </Button>
  )
}

function SidebarRail({ className, ...props }: React.ComponentProps<"button">) {
  const { toggleSidebar } = useSidebar()

  return (
    <button
      data-sidebar="rail"
      data-slot="sidebar-rail"
      aria-label="Toggle Sidebar"
      tabIndex={-1}
      onClick={toggleSidebar}
      title="Toggle Sidebar"
      className={cn(
        "hover:after:bg-sidebar-border absolute inset-y-0 z-20 hidden w-4 -translate-x-1/2 transition-all ease-linear group-data-[side=left]:-right-4 group-data-[side=right]:left-0 after:absolute after:inset-y-0 after:left-1/2 after:w-[2px] sm:flex",
        "in-data-[side=left]:cursor-w-resize in-data-[side=right]:cursor-e-resize",
        "[[data-side=left][data-state=collapsed]_&]:cursor-e-resize [[data-side=right][data-state=collapsed]_&]:cursor-w-resize",
        "hover:group-data-[collapsible=offcanvas]:bg-sidebar group-data-[collapsible=offcanvas]:translate-x-0 group-data-[collapsible=offcanvas]:after:left-full",
        "[[data-side=left][data-collapsible=offcanvas]_&]:-right-2",
        "[[data-side=right][data-collapsible=offcanvas]_&]:-left-2",
        className
      )}
      {...props}
    />
  )
}

function SidebarInset({ className, ...props }: React.ComponentProps<"main">) {
  return (
    <main
      data-slot="sidebar-inset"
      className={cn(
        "bg-background relative flex w-full flex-1 flex-col",
        "md:peer-data-[variant=inset]:m-2 md:peer-data-[variant=inset]:ml-0 md:peer-data-[variant=inset]:rounded-xl md:peer-data-[variant=inset]:shadow-sm md:peer-data-[variant=inset]:peer-data-[state=collapsed]:ml-2",
        className
      )}
      {...props}
    />
  )
}

function SidebarInput({
  className,
  ...props
}: React.ComponentProps<typeof Input>) {
  return (
    <Input
      data-slot="sidebar-input"
      data-sidebar="input"
      className={cn("bg-background h-8 w-full shadow-none", className)}
      {...props}
    />
  )
}

function SidebarHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-header"
      data-sidebar="header"
      className={cn("flex flex-col gap-2 p-2", className)}
      {...props}
    />
  )
}

function SidebarFooter({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-footer"
      data-sidebar="footer"
      className={cn("flex flex-col gap-2 p-2", className)}
      {...props}
    />
  )
}

function SidebarSeparator({
  className,
  ...props
}: React.ComponentProps<typeof Separator>) {
  return (
    <Separator
      data-slot="sidebar-separator"
      data-sidebar="separator"
      className={cn("bg-sidebar-border mx-2 w-auto", className)}
      {...props}
    />
  )
}

function SidebarContent({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-content"
      data-sidebar="content"
      className={cn(
        "flex min-h-0 flex-1 flex-col gap-2 overflow-auto group-data-[collapsible=icon]:overflow-hidden",
        className
      )}
      {...props}
    />
  )
}

function SidebarGroup({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-group"
      data-sidebar="group"
      className={cn("relative flex w-full min-w-0 flex-col p-2", className)}
      {...props}
    />
  )
}

function SidebarGroupLabel({
  className,
  asChild = false,
  ...props
}: React.ComponentProps<"div"> & { asChild?: boolean }) {
  const Comp = asChild ? SlotPrimitive.Slot : "div"

  return (
    <Comp
      data-slot="sidebar-group-label"
      data-sidebar="group-label"
      className={cn(
        "text-sidebar-foreground/70 ring-sidebar-ring flex h-8 shrink-0 items-center rounded-md px-2 text-xs font-medium outline-hidden transition-[margin,opacity] duration-200 ease-linear focus-visible:ring-2 [&>svg]:size-4 [&>svg]:shrink-0",
        "group-data-[collapsible=icon]:-mt-8 group-data-[collapsible=icon]:opacity-0",
        className
      )}
      {...props}
    />
  )
}

function SidebarGroupAction({
  className,
  asChild = false,
  ...props
}: React.ComponentProps<"button"> & { asChild?: boolean }) {
  const Comp = asChild ? SlotPrimitive.Slot : "button"

  return (
    <Comp
      data-slot="sidebar-group-action"
      data-sidebar="group-action"
      className={cn(
        "text-sidebar-foreground ring-sidebar-ring hover:bg-sidebar-accent hover:text-sidebar-accent-foreground absolute top-3.5 right-3 flex aspect-square w-5 items-center justify-center rounded-md p-0 outline-hidden transition-transform focus-visible:ring-2 [&>svg]:size-4 [&>svg]:shrink-0",
        // Increases the hit area of the button on mobile.
        "after:absolute after:-inset-2 md:after:hidden",
        "group-data-[collapsible=icon]:hidden",
        className
      )}
      {...props}
    />
  )
}

function SidebarGroupContent({
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-group-content"
      data-sidebar="group-content"
      className={cn("w-full text-sm", className)}
      {...props}
    />
  )
}

function SidebarMenu({ className, ...props }: React.ComponentProps<"ul">) {
  return (
    <ul
      data-slot="sidebar-menu"
      data-sidebar="menu"
      className={cn("flex w-full min-w-0 flex-col gap-1", className)}
      {...props}
    />
  )
}

function SidebarMenuItem({ className, ...props }: React.ComponentProps<"li">) {
  return (
    <li
      data-slot="sidebar-menu-item"
      data-sidebar="menu-item"
      className={cn("group/menu-item relative", className)}
      {...props}
    />
  )
}

const sidebarMenuButtonVariants = cva(
  "peer/menu-button flex w-full items-center gap-2 overflow-hidden rounded-md p-2 text-left text-sm outline-hidden ring-sidebar-ring transition-[width,height,padding] hover:bg-sidebar-accent hover:text-sidebar-accent-foreground focus-visible:ring-2 active:bg-sidebar-accent active:text-sidebar-accent-foreground disabled:pointer-events-none disabled:opacity-50 group-has-data-[sidebar=menu-action]/menu-item:pr-8 aria-disabled:pointer-events-none aria-disabled:opacity-50 data-[active=true]:bg-sidebar-accent data-[active=true]:font-medium data-[active=true]:text-sidebar-accent-foreground data-[state=open]:hover:bg-sidebar-accent data-[state=open]:hover:text-sidebar-accent-foreground group-data-[collapsible=icon]:size-8! group-data-[collapsible=icon]:p-2! [&>span:last-child]:truncate [&>svg]:size-4 [&>svg]:shrink-0",
  {
    variants: {
      variant: {
        default: "hover:bg-sidebar-accent hover:text-sidebar-accent-foreground",
        outline:
          "bg-background shadow-[0_0_0_1px_hsl(var(--sidebar-border))] hover:bg-sidebar-accent hover:text-sidebar-accent-foreground hover:shadow-[0_0_0_1px_hsl(var(--sidebar-accent))]",
      },
      size: {
        default: "h-8 text-sm",
        sm: "h-7 text-xs",
        lg: "h-12 text-sm group-data-[collapsible=icon]:p-0!",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function SidebarMenuButton({
  asChild = false,
  isActive = false,
  variant = "default",
  size = "default",
  tooltip,
  className,
  ...props
}: React.ComponentProps<"button"> & {
  asChild?: boolean
  isActive?: boolean
  tooltip?: string | React.ComponentProps<typeof TooltipContent>
} & VariantProps<typeof sidebarMenuButtonVariants>) {
  const Comp = asChild ? SlotPrimitive.Slot : "button"
  const { isMobile, state } = useSidebar()

  const button = (
    <Comp
      data-slot="sidebar-menu-button"
      data-sidebar="menu-button"
      data-size={size}
      data-active={isActive}
      className={cn(sidebarMenuButtonVariants({ variant, size }), className)}
      {...props}
    />
  )

  if (!tooltip) {
    return button
  }

  if (typeof tooltip === "string") {
    tooltip = {
      children: tooltip,
    }
  }

  return (
    <Tooltip>
      <TooltipTrigger asChild>{button}</TooltipTrigger>
      <TooltipContent
        side="right"
        align="center"
        hidden={state !== "collapsed" || isMobile}
        {...tooltip}
      />
    </Tooltip>
  )
}

function SidebarMenuAction({
  className,
  asChild = false,
  showOnHover = false,
  ...props
}: React.ComponentProps<"button"> & {
  asChild?: boolean
  showOnHover?: boolean
}) {
  const Comp = asChild ? SlotPrimitive.Slot : "button"

  return (
    <Comp
      data-slot="sidebar-menu-action"
      data-sidebar="menu-action"
      className={cn(
        "text-sidebar-foreground ring-sidebar-ring hover:bg-sidebar-accent hover:text-sidebar-accent-foreground peer-hover/menu-button:text-sidebar-accent-foreground absolute top-1.5 right-1 flex aspect-square w-5 items-center justify-center rounded-md p-0 outline-hidden transition-transform focus-visible:ring-2 [&>svg]:size-4 [&>svg]:shrink-0",
        // Increases the hit area of the button on mobile.
        "after:absolute after:-inset-2 md:after:hidden",
        "peer-data-[size=sm]/menu-button:top-1",
        "peer-data-[size=default]/menu-button:top-1.5",
        "peer-data-[size=lg]/menu-button:top-2.5",
        "group-data-[collapsible=icon]:hidden",
        showOnHover &&
          "peer-data-[active=true]/menu-button:text-sidebar-accent-foreground group-focus-within/menu-item:opacity-100 group-hover/menu-item:opacity-100 data-[state=open]:opacity-100 md:opacity-0",
        className
      )}
      {...props}
    />
  )
}

function SidebarMenuBadge({
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="sidebar-menu-badge"
      data-sidebar="menu-badge"
      className={cn(
        "text-sidebar-foreground pointer-events-none absolute right-1 flex h-5 min-w-5 items-center justify-center rounded-md px-1 text-xs font-medium tabular-nums select-none",
        "peer-hover/menu-button:text-sidebar-accent-foreground peer-data-[active=true]/menu-button:text-sidebar-accent-foreground",
        "peer-data-[size=sm]/menu-button:top-1",
        "peer-data-[size=default]/menu-button:top-1.5",
        "peer-data-[size=lg]/menu-button:top-2.5",
        "group-data-[collapsible=icon]:hidden",
        className
      )}
      {...props}
    />
  )
}

function SidebarMenuSkeleton({
  className,
  showIcon = false,
  ...props
}: React.ComponentProps<"div"> & {
  showIcon?: boolean
}) {
  // Random width between 50 to 90%.
  const width = React.useMemo(() => {
    return `${Math.floor(Math.random() * 40) + 50}%`
  }, [])

  return (
    <div
      data-slot="sidebar-menu-skeleton"
      data-sidebar="menu-skeleton"
      className={cn("flex h-8 items-center gap-2 rounded-md px-2", className)}
      {...props}
    >
      {showIcon && (
        <Skeleton
          className="size-4 rounded-md"
          data-sidebar="menu-skeleton-icon"
        />
      )}
      <Skeleton
        className="h-4 max-w-(--skeleton-width) flex-1"
        data-sidebar="menu-skeleton-text"
        style={
          {
            "--skeleton-width": width,
          } as React.CSSProperties
        }
      />
    </div>
  )
}

function SidebarMenuSub({ className, ...props }: React.ComponentProps<"ul">) {
  return (
    <ul
      data-slot="sidebar-menu-sub"
      data-sidebar="menu-sub"
      className={cn(
        "border-sidebar-border mx-3.5 flex min-w-0 translate-x-px flex-col gap-1 border-l px-2.5 py-0.5",
        "group-data-[collapsible=icon]:hidden",
        className
      )}
      {...props}
    />
  )
}

function SidebarMenuSubItem({
  className,
  ...props
}: React.ComponentProps<"li">) {
  return (
    <li
      data-slot="sidebar-menu-sub-item"
      data-sidebar="menu-sub-item"
      className={cn("group/menu-sub-item relative", className)}
      {...props}
    />
  )
}

function SidebarMenuSubButton({
  asChild = false,
  size = "md",
  isActive = false,
  className,
  ...props
}: React.ComponentProps<"a"> & {
  asChild?: boolean
  size?: "sm" | "md"
  isActive?: boolean
}) {
  const Comp = asChild ? SlotPrimitive.Slot : "a"

  return (
    <Comp
      data-slot="sidebar-menu-sub-button"
      data-sidebar="menu-sub-button"
      data-size={size}
      data-active={isActive}
      className={cn(
        "text-sidebar-foreground ring-sidebar-ring hover:bg-sidebar-accent hover:text-sidebar-accent-foreground active:bg-sidebar-accent active:text-sidebar-accent-foreground [&>svg]:text-sidebar-accent-foreground flex h-7 min-w-0 -translate-x-px items-center gap-2 overflow-hidden rounded-md px-2 outline-hidden focus-visible:ring-2 disabled:pointer-events-none disabled:opacity-50 aria-disabled:pointer-events-none aria-disabled:opacity-50 [&>span:last-child]:truncate [&>svg]:size-4 [&>svg]:shrink-0",
        "data-[active=true]:bg-sidebar-accent data-[active=true]:text-sidebar-accent-foreground",
        size === "sm" && "text-xs",
        size === "md" && "text-sm",
        "group-data-[collapsible=icon]:hidden",
        className
      )}
      {...props}
    />
  )
}

export {
  Sidebar,
  SidebarContent,
  SidebarFooter,
  SidebarGroup,
  SidebarGroupAction,
  SidebarGroupContent,
  SidebarGroupLabel,
  SidebarHeader,
  SidebarInput,
  SidebarInset,
  SidebarMenu,
  SidebarMenuAction,
  SidebarMenuBadge,
  SidebarMenuButton,
  SidebarMenuItem,
  SidebarMenuSkeleton,
  SidebarMenuSub,
  SidebarMenuSubButton,
  SidebarMenuSubItem,
  SidebarProvider,
  SidebarRail,
  SidebarSeparator,
  SidebarTrigger,
  useSidebar,
}
````

## File: components/ui/skeleton.tsx
````typescript
import { cn } from "@/lib/utils"

function Skeleton({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="skeleton"
      className={cn("bg-accent animate-pulse rounded-md", className)}
      {...props}
    />
  )
}

export { Skeleton }
````

## File: components/ui/slider.tsx
````typescript
"use client"

import * as React from "react"
import { Slider as SliderPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Slider({
  className,
  defaultValue,
  value,
  min = 0,
  max = 100,
  ...props
}: React.ComponentProps<typeof SliderPrimitive.Root>) {
  const _values = React.useMemo(
    () =>
      Array.isArray(value)
        ? value
        : Array.isArray(defaultValue)
          ? defaultValue
          : [min, max],
    [value, defaultValue, min, max]
  )

  return (
    <SliderPrimitive.Root
      data-slot="slider"
      defaultValue={defaultValue}
      value={value}
      min={min}
      max={max}
      className={cn(
        "relative flex w-full touch-none items-center select-none data-[disabled]:opacity-50 data-[orientation=vertical]:h-full data-[orientation=vertical]:min-h-44 data-[orientation=vertical]:w-auto data-[orientation=vertical]:flex-col",
        className
      )}
      {...props}
    >
      <SliderPrimitive.Track
        data-slot="slider-track"
        className={cn(
          "bg-muted relative grow overflow-hidden rounded-full data-[orientation=horizontal]:h-1.5 data-[orientation=horizontal]:w-full data-[orientation=vertical]:h-full data-[orientation=vertical]:w-1.5"
        )}
      >
        <SliderPrimitive.Range
          data-slot="slider-range"
          className={cn(
            "bg-primary absolute data-[orientation=horizontal]:h-full data-[orientation=vertical]:w-full"
          )}
        />
      </SliderPrimitive.Track>
      {Array.from({ length: _values.length }, (_, index) => (
        <SliderPrimitive.Thumb
          data-slot="slider-thumb"
          key={index}
          className="border-primary ring-ring/50 block size-4 shrink-0 rounded-full border bg-white shadow-sm transition-[color,box-shadow] hover:ring-4 focus-visible:ring-4 focus-visible:outline-hidden disabled:pointer-events-none disabled:opacity-50"
        />
      ))}
    </SliderPrimitive.Root>
  )
}

export { Slider }
````

## File: components/ui/sonner.tsx
````typescript
"use client"

import {
  CircleCheckIcon,
  InfoIcon,
  Loader2Icon,
  OctagonXIcon,
  TriangleAlertIcon,
} from "lucide-react"
import { useTheme } from "next-themes"
import { Toaster as Sonner, type ToasterProps } from "sonner"

const Toaster = ({ ...props }: ToasterProps) => {
  const { theme = "system" } = useTheme()

  return (
    <Sonner
      theme={theme as ToasterProps["theme"]}
      className="toaster group"
      icons={{
        success: <CircleCheckIcon className="size-4" />,
        info: <InfoIcon className="size-4" />,
        warning: <TriangleAlertIcon className="size-4" />,
        error: <OctagonXIcon className="size-4" />,
        loading: <Loader2Icon className="size-4 animate-spin" />,
      }}
      style={
        {
          "--normal-bg": "var(--popover)",
          "--normal-text": "var(--popover-foreground)",
          "--normal-border": "var(--border)",
          "--border-radius": "var(--radius)",
        } as React.CSSProperties
      }
      {...props}
    />
  )
}

export { Toaster }
````

## File: components/ui/spinner.tsx
````typescript
import { Loader2Icon } from "lucide-react"

import { cn } from "@/lib/utils"

function Spinner({ className, ...props }: React.ComponentProps<"svg">) {
  return (
    <Loader2Icon
      role="status"
      aria-label="Loading"
      className={cn("size-4 animate-spin", className)}
      {...props}
    />
  )
}

export { Spinner }
````

## File: components/ui/switch.tsx
````typescript
"use client"

import * as React from "react"
import { Switch as SwitchPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Switch({
  className,
  ...props
}: React.ComponentProps<typeof SwitchPrimitive.Root>) {
  return (
    <SwitchPrimitive.Root
      data-slot="switch"
      className={cn(
        "peer data-[state=checked]:bg-primary data-[state=unchecked]:bg-input focus-visible:border-ring focus-visible:ring-ring/50 dark:data-[state=unchecked]:bg-input/80 inline-flex h-[1.15rem] w-8 shrink-0 items-center rounded-full border border-transparent shadow-xs transition-all outline-none focus-visible:ring-[3px] disabled:cursor-not-allowed disabled:opacity-50",
        className
      )}
      {...props}
    >
      <SwitchPrimitive.Thumb
        data-slot="switch-thumb"
        className={cn(
          "bg-background dark:data-[state=unchecked]:bg-foreground dark:data-[state=checked]:bg-primary-foreground pointer-events-none block size-4 rounded-full ring-0 transition-transform data-[state=checked]:translate-x-[calc(100%-2px)] data-[state=unchecked]:translate-x-0"
        )}
      />
    </SwitchPrimitive.Root>
  )
}

export { Switch }
````

## File: components/ui/table.tsx
````typescript
"use client"

import * as React from "react"

import { cn } from "@/lib/utils"

function Table({ className, ...props }: React.ComponentProps<"table">) {
  return (
    <div
      data-slot="table-container"
      className="relative w-full overflow-x-auto"
    >
      <table
        data-slot="table"
        className={cn("w-full caption-bottom text-sm", className)}
        {...props}
      />
    </div>
  )
}

function TableHeader({ className, ...props }: React.ComponentProps<"thead">) {
  return (
    <thead
      data-slot="table-header"
      className={cn("[&_tr]:border-b", className)}
      {...props}
    />
  )
}

function TableBody({ className, ...props }: React.ComponentProps<"tbody">) {
  return (
    <tbody
      data-slot="table-body"
      className={cn("[&_tr:last-child]:border-0", className)}
      {...props}
    />
  )
}

function TableFooter({ className, ...props }: React.ComponentProps<"tfoot">) {
  return (
    <tfoot
      data-slot="table-footer"
      className={cn(
        "bg-muted/50 border-t font-medium [&>tr]:last:border-b-0",
        className
      )}
      {...props}
    />
  )
}

function TableRow({ className, ...props }: React.ComponentProps<"tr">) {
  return (
    <tr
      data-slot="table-row"
      className={cn(
        "hover:bg-muted/50 data-[state=selected]:bg-muted border-b transition-colors",
        className
      )}
      {...props}
    />
  )
}

function TableHead({ className, ...props }: React.ComponentProps<"th">) {
  return (
    <th
      data-slot="table-head"
      className={cn(
        "text-foreground h-10 px-2 text-left align-middle font-medium whitespace-nowrap [&:has([role=checkbox])]:pr-0 [&>[role=checkbox]]:translate-y-[2px]",
        className
      )}
      {...props}
    />
  )
}

function TableCell({ className, ...props }: React.ComponentProps<"td">) {
  return (
    <td
      data-slot="table-cell"
      className={cn(
        "p-2 align-middle whitespace-nowrap [&:has([role=checkbox])]:pr-0 [&>[role=checkbox]]:translate-y-[2px]",
        className
      )}
      {...props}
    />
  )
}

function TableCaption({
  className,
  ...props
}: React.ComponentProps<"caption">) {
  return (
    <caption
      data-slot="table-caption"
      className={cn("text-muted-foreground mt-4 text-sm", className)}
      {...props}
    />
  )
}

export {
  Table,
  TableHeader,
  TableBody,
  TableFooter,
  TableHead,
  TableRow,
  TableCell,
  TableCaption,
}
````

## File: components/ui/tabs.tsx
````typescript
"use client"

import * as React from "react"
import { Tabs as TabsPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function Tabs({
  className,
  ...props
}: React.ComponentProps<typeof TabsPrimitive.Root>) {
  return (
    <TabsPrimitive.Root
      data-slot="tabs"
      className={cn("flex flex-col gap-2", className)}
      {...props}
    />
  )
}

function TabsList({
  className,
  ...props
}: React.ComponentProps<typeof TabsPrimitive.List>) {
  return (
    <TabsPrimitive.List
      data-slot="tabs-list"
      className={cn(
        "bg-muted text-muted-foreground inline-flex h-9 w-fit items-center justify-center rounded-lg p-[3px]",
        className
      )}
      {...props}
    />
  )
}

function TabsTrigger({
  className,
  ...props
}: React.ComponentProps<typeof TabsPrimitive.Trigger>) {
  return (
    <TabsPrimitive.Trigger
      data-slot="tabs-trigger"
      className={cn(
        "data-[state=active]:bg-background dark:data-[state=active]:text-foreground focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:outline-ring dark:data-[state=active]:border-input dark:data-[state=active]:bg-input/30 text-foreground dark:text-muted-foreground inline-flex h-[calc(100%-1px)] flex-1 items-center justify-center gap-1.5 rounded-md border border-transparent px-2 py-1 text-sm font-medium whitespace-nowrap transition-[color,box-shadow] focus-visible:ring-[3px] focus-visible:outline-1 disabled:pointer-events-none disabled:opacity-50 data-[state=active]:shadow-sm [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
        className
      )}
      {...props}
    />
  )
}

function TabsContent({
  className,
  ...props
}: React.ComponentProps<typeof TabsPrimitive.Content>) {
  return (
    <TabsPrimitive.Content
      data-slot="tabs-content"
      className={cn("flex-1 outline-none", className)}
      {...props}
    />
  )
}

export { Tabs, TabsList, TabsTrigger, TabsContent }
````

## File: components/ui/textarea.tsx
````typescript
import * as React from "react"

import { cn } from "@/lib/utils"

function Textarea({ className, ...props }: React.ComponentProps<"textarea">) {
  return (
    <textarea
      data-slot="textarea"
      className={cn(
        "border-input placeholder:text-muted-foreground focus-visible:border-ring focus-visible:ring-ring/50 aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive dark:bg-input/30 flex field-sizing-content min-h-16 w-full rounded-md border bg-transparent px-3 py-2 text-base shadow-xs transition-[color,box-shadow] outline-none focus-visible:ring-[3px] disabled:cursor-not-allowed disabled:opacity-50 md:text-sm",
        className
      )}
      {...props}
    />
  )
}

export { Textarea }
````

## File: components/ui/toggle-group.tsx
````typescript
"use client"

import * as React from "react"
import { ToggleGroup as ToggleGroupPrimitive } from "radix-ui"
import { type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"
import { toggleVariants } from "@/components/ui/toggle"

const ToggleGroupContext = React.createContext<
  VariantProps<typeof toggleVariants> & {
    spacing?: number
  }
>({
  size: "default",
  variant: "default",
  spacing: 0,
})

function ToggleGroup({
  className,
  variant,
  size,
  spacing = 0,
  children,
  ...props
}: React.ComponentProps<typeof ToggleGroupPrimitive.Root> &
  VariantProps<typeof toggleVariants> & {
    spacing?: number
  }) {
  return (
    <ToggleGroupPrimitive.Root
      data-slot="toggle-group"
      data-variant={variant}
      data-size={size}
      data-spacing={spacing}
      style={{ "--gap": spacing } as React.CSSProperties}
      className={cn(
        "group/toggle-group flex w-fit items-center gap-[--spacing(var(--gap))] rounded-md data-[spacing=default]:data-[variant=outline]:shadow-xs",
        className
      )}
      {...props}
    >
      <ToggleGroupContext.Provider value={{ variant, size, spacing }}>
        {children}
      </ToggleGroupContext.Provider>
    </ToggleGroupPrimitive.Root>
  )
}

function ToggleGroupItem({
  className,
  children,
  variant,
  size,
  ...props
}: React.ComponentProps<typeof ToggleGroupPrimitive.Item> &
  VariantProps<typeof toggleVariants>) {
  const context = React.useContext(ToggleGroupContext)

  return (
    <ToggleGroupPrimitive.Item
      data-slot="toggle-group-item"
      data-variant={context.variant || variant}
      data-size={context.size || size}
      data-spacing={context.spacing}
      className={cn(
        toggleVariants({
          variant: context.variant || variant,
          size: context.size || size,
        }),
        "w-auto min-w-0 shrink-0 px-3 focus:z-10 focus-visible:z-10",
        "data-[spacing=0]:rounded-none data-[spacing=0]:shadow-none data-[spacing=0]:first:rounded-l-md data-[spacing=0]:last:rounded-r-md data-[spacing=0]:data-[variant=outline]:border-l-0 data-[spacing=0]:data-[variant=outline]:first:border-l",
        className
      )}
      {...props}
    >
      {children}
    </ToggleGroupPrimitive.Item>
  )
}

export { ToggleGroup, ToggleGroupItem }
````

## File: components/ui/toggle.tsx
````typescript
"use client"

import * as React from "react"
import { Toggle as TogglePrimitive } from "radix-ui"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const toggleVariants = cva(
  "inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium hover:bg-muted hover:text-muted-foreground disabled:pointer-events-none disabled:opacity-50 data-[state=on]:bg-accent data-[state=on]:text-accent-foreground [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4 [&_svg]:shrink-0 focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px] outline-none transition-[color,box-shadow] aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive whitespace-nowrap",
  {
    variants: {
      variant: {
        default: "bg-transparent",
        outline:
          "border border-input bg-transparent shadow-xs hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-9 px-2 min-w-9",
        sm: "h-8 px-1.5 min-w-8",
        lg: "h-10 px-2.5 min-w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

function Toggle({
  className,
  variant,
  size,
  ...props
}: React.ComponentProps<typeof TogglePrimitive.Root> &
  VariantProps<typeof toggleVariants>) {
  return (
    <TogglePrimitive.Root
      data-slot="toggle"
      className={cn(toggleVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Toggle, toggleVariants }
````

## File: components/ui/tooltip.tsx
````typescript
"use client"

import * as React from "react"
import { Tooltip as TooltipPrimitive } from "radix-ui"

import { cn } from "@/lib/utils"

function TooltipProvider({
  delayDuration = 0,
  ...props
}: React.ComponentProps<typeof TooltipPrimitive.Provider>) {
  return (
    <TooltipPrimitive.Provider
      data-slot="tooltip-provider"
      delayDuration={delayDuration}
      {...props}
    />
  )
}

function Tooltip({
  ...props
}: React.ComponentProps<typeof TooltipPrimitive.Root>) {
  return (
    <TooltipProvider>
      <TooltipPrimitive.Root data-slot="tooltip" {...props} />
    </TooltipProvider>
  )
}

function TooltipTrigger({
  ...props
}: React.ComponentProps<typeof TooltipPrimitive.Trigger>) {
  return <TooltipPrimitive.Trigger data-slot="tooltip-trigger" {...props} />
}

function TooltipContent({
  className,
  sideOffset = 0,
  children,
  ...props
}: React.ComponentProps<typeof TooltipPrimitive.Content>) {
  return (
    <TooltipPrimitive.Portal>
      <TooltipPrimitive.Content
        data-slot="tooltip-content"
        sideOffset={sideOffset}
        className={cn(
          "bg-foreground text-background animate-in fade-in-0 zoom-in-95 data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=closed]:zoom-out-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-50 w-fit origin-(--radix-tooltip-content-transform-origin) rounded-md px-3 py-1.5 text-xs text-balance",
          className
        )}
        {...props}
      >
        {children}
        <TooltipPrimitive.Arrow className="bg-foreground fill-foreground z-50 size-2.5 translate-y-[calc(-50%_-_2px)] rotate-45 rounded-[2px]" />
      </TooltipPrimitive.Content>
    </TooltipPrimitive.Portal>
  )
}

export { Tooltip, TooltipTrigger, TooltipContent, TooltipProvider }
````

## File: components/chart-loading.tsx
````typescript
"use client";

import { AreaChart, CartesianGrid, XAxis } from "recharts";
import { Card, CardContent } from "@/components/ui/card";
import {
  ChartContainer,
  ChartLegend,
  ChartLegendContent,
  ChartTooltip,
  ChartTooltipContent,
} from "@/components/ui/chart";

const mockData = Array.from({ length: 30 }, (_, index) => ({
  date: new Date(Date.now() - index * 24 * 60 * 60 * 1000).toISOString(),
  downloads: Math.floor(Math.random() * 1000),
}));

export const ChartLoading = () => (
  <Card className="size-full shadow-none">
    <CardContent className="size-full">
      <ChartContainer className="size-full" config={{}}>
        <AreaChart data={mockData}>
          <defs>
            <linearGradient id="fillDownloads" x1="0" x2="0" y1="0" y2="1">
              <stop
                offset="5%"
                stopColor="var(--color-border)"
                stopOpacity={0.8}
              />
              <stop
                offset="95%"
                stopColor="var(--color-border)"
                stopOpacity={0.1}
              />
            </linearGradient>
          </defs>
          <CartesianGrid vertical={false} />
          <XAxis
            axisLine={false}
            dataKey="date"
            minTickGap={32}
            tickFormatter={(value) => {
              const date = new Date(value);
              return date.toLocaleDateString("en-US", {
                month: "short",
                day: "numeric",
              });
            }}
            tickLine={false}
            tickMargin={8}
          />
          <ChartTooltip
            content={
              <ChartTooltipContent
                indicator="dot"
                labelFormatter={(value) =>
                  new Date(value).toLocaleDateString("en-US", {
                    month: "short",
                    day: "numeric",
                  })
                }
              />
            }
            cursor={false}
          />
          <ChartLegend content={<ChartLegendContent />} />
        </AreaChart>
      </ChartContainer>
    </CardContent>
  </Card>
);
````

## File: components/chart.tsx
````typescript
"use client";

import { addMonths, addWeeks } from "date-fns";
import { useEffect, useMemo } from "react";
import { CartesianGrid, Line, LineChart, XAxis, YAxis } from "recharts";
import { DownloadTooltip } from "@/components/download-tooltip";
import { Card, CardContent } from "@/components/ui/card";
import {
  ChartContainer,
  ChartLegend,
  ChartLegendContent,
  ChartTooltip,
} from "@/components/ui/chart";
import {
  Tooltip,
  TooltipContent,
  TooltipTrigger,
} from "@/components/ui/tooltip";
import { formatNumber, shouldRemoveIncompleteDate } from "@/lib/chart-utils";
import { prepareDownloadData } from "@/lib/download-data";
import { useGrouping, useMetric, useZeroMode } from "@/providers/filters";

export const description = "An interactive line chart";

const colors = [
  "var(--chart-1)",
  "var(--chart-2)",
  "var(--chart-3)",
  "var(--chart-4)",
  "var(--chart-5)",
];

interface ChartAreaInteractiveProps {
  data: {
    start: string;
    end: string;
    package: string;
    downloads: {
      downloads: number;
      day: string;
    }[];
  }[];
}

const getDateRangeEnd = (startDate: string, grouping: string) => {
  const date = new Date(startDate);

  if (grouping === "week") {
    return addWeeks(date, 1);
  }

  if (grouping === "month") {
    return addMonths(date, 1);
  }

  return date;
};

export const ChartAreaInteractive = ({ data }: ChartAreaInteractiveProps) => {
  const [grouping] = useGrouping();
  const [metric, setMetric] = useMetric();
  const [zeroMode] = useZeroMode();
  const preparedByPackage = useMemo(
    () =>
      new Map(
        data.map((pkg) => [
          pkg.package,
          new Map(
            prepareDownloadData(pkg.downloads, grouping, zeroMode).map(
              (point) => [point.date, point]
            )
          ),
        ])
      ),
    [data, grouping, zeroMode]
  );
  const isShare = metric === "share" && data.length > 1;
  const packageNames = useMemo(() => data.map((pkg) => pkg.package), [data]);

  useEffect(() => {
    if (metric === "share" && data.length <= 1) {
      setMetric(null);
    }
  }, [metric, data.length, setMetric]);

  const chartData = useMemo(() => {
    const allDates = new Set<string>();
    const packagesByDate: Record<string, Record<string, number>> = {};

    for (const pkg of data) {
      const grouped = preparedByPackage.get(pkg.package);
      for (const item of grouped?.values() ?? []) {
        allDates.add(item.date);
        if (!packagesByDate[item.date]) {
          packagesByDate[item.date] = {};
        }
        packagesByDate[item.date][pkg.package] = item.downloads;
      }
    }

    const sortedDates = Array.from(allDates).sort();

    // Remove the most recent data point to avoid showing incomplete periods
    if (sortedDates.length > 1) {
      const lastDateString = sortedDates.at(-1);
      if (
        lastDateString &&
        shouldRemoveIncompleteDate(lastDateString, grouping)
      ) {
        sortedDates.pop();
      }
    }

    return sortedDates.map((date) => {
      const row: Record<string, string | number> = {
        date,
        dateEnd: getDateRangeEnd(date, grouping).toISOString().split("T")[0],
      };

      const dateData = packagesByDate[date] ?? {};

      if (isShare) {
        const total = packageNames.reduce(
          (sum, name) => sum + (dateData[name] ?? 0),
          0
        );
        for (const name of packageNames) {
          row[name] =
            total > 0
              ? Number((((dateData[name] ?? 0) / total) * 100).toFixed(1))
              : 0;
        }
      } else {
        for (const name of packageNames) {
          row[name] = dateData[name] ?? 0;
        }
      }

      return row;
    });
  }, [data, grouping, isShare, packageNames, preparedByPackage]);

  const hasEstimates = chartData.some((row) =>
    packageNames.some(
      (name) =>
        (preparedByPackage.get(name)?.get(String(row.date))?.estimatedDays ??
          0) > 0
    )
  );

  const shareYMax = useMemo(() => {
    if (!isShare || chartData.length === 0) {
      return 100;
    }
    const maxValue = Math.max(
      ...chartData.flatMap((row) =>
        typeof row === "string"
          ? [0]
          : packageNames.map((name) => Number(row[name] ?? 0))
      )
    );
    return Math.min(100, Math.ceil((maxValue + 5) / 10) * 10);
  }, [isShare, chartData, packageNames]);

  const chartConfig = data.reduce(
    (acc, pkg, index) => {
      acc[pkg.package] = {
        label: pkg.package,
        color: colors[index % colors.length],
      };
      return acc;
    },
    {} as Record<string, { label: string; color: string }>
  );

  return (
    <Card className="size-full shadow-none">
      <CardContent className="min-h-0 flex-1">
        <ChartContainer className="size-full" config={chartConfig}>
          <LineChart data={chartData}>
            <CartesianGrid vertical={false} />
            <XAxis
              axisLine={false}
              dataKey="date"
              minTickGap={32}
              tickFormatter={(value) => {
                const [year, month, day] = value.split("-");
                const date = new Date(
                  Number(year),
                  Number(month) - 1,
                  Number(day)
                );
                return date.toLocaleDateString("en-US", {
                  month: "short",
                  day: "numeric",
                });
              }}
              tickLine={false}
              tickMargin={8}
            />
            <YAxis
              axisLine={false}
              domain={isShare ? [0, shareYMax] : undefined}
              tickFormatter={(value) => {
                if (isShare) {
                  return `${value}%`;
                }
                return formatNumber(value);
              }}
              tickLine={false}
              tickMargin={8}
            />
            <ChartTooltip
              content={
                <DownloadTooltip
                  grouping={grouping}
                  isShare={isShare}
                  preparedByPackage={preparedByPackage}
                />
              }
              cursor={false}
            />
            {data.map((pkg, index) => (
              <Line
                dataKey={pkg.package}
                dot={({ cx, cy, payload }) => {
                  const point = preparedByPackage
                    .get(pkg.package)
                    ?.get(payload.date);
                  const estimated = (point?.estimatedDays ?? 0) > 0;
                  const denominatorEstimated =
                    isShare &&
                    packageNames.some(
                      (name) =>
                        (preparedByPackage.get(name)?.get(payload.date)
                          ?.estimatedDays ?? 0) > 0
                    );
                  return estimated || denominatorEstimated ? (
                    <circle
                      cx={cx}
                      cy={cy}
                      data-estimated-date={payload.date}
                      data-estimated-package={pkg.package}
                      fill="var(--background)"
                      key={`${pkg.package}-${payload.date}`}
                      r={4}
                      stroke={colors[index % colors.length]}
                      strokeDasharray="2 2"
                      strokeWidth={1.5}
                    >
                      <title>
                        {pkg.package}:{" "}
                        {estimated
                          ? "estimated downloads"
                          : "share includes estimates"}
                      </title>
                    </circle>
                  ) : (
                    <g key={`${pkg.package}-${payload.date}`} />
                  );
                }}
                key={pkg.package}
                stroke={colors[index % colors.length]}
                strokeWidth={2}
                type="monotone"
              />
            ))}
            <ChartLegend
              content={(props) => (
                <div className="flex flex-wrap items-center justify-center gap-x-4 gap-y-1 pt-3">
                  <ChartLegendContent
                    className="pt-0"
                    payload={props.payload}
                    verticalAlign={props.verticalAlign}
                  />
                  {hasEstimates && (
                    <Tooltip>
                      <TooltipTrigger asChild>
                        <button
                          aria-label="About estimated downloads"
                          className="inline-flex items-center gap-1 rounded-sm text-muted-foreground text-xs outline-none focus-visible:ring-2 focus-visible:ring-ring"
                          type="button"
                        >
                          <span aria-hidden="true">≈</span>
                          <span className="hidden sm:inline">Estimated</span>
                        </button>
                      </TooltipTrigger>
                      <TooltipContent className="max-w-64" sideOffset={6}>
                        Short zero-download gaps are estimated. Hover a point to
                        compare with npm-reported counts.
                        {isShare && " Shares include estimated downloads."}
                      </TooltipContent>
                    </Tooltip>
                  )}
                </div>
              )}
            />
          </LineChart>
        </ChartContainer>
      </CardContent>
    </Card>
  );
};
````

## File: components/download-tooltip.tsx
````typescript
"use client";

import type { ComponentProps } from "react";
import { ChartTooltipContent } from "@/components/ui/chart";
import type { PreparedDownload } from "@/lib/download-data";

type DownloadTooltipProps = ComponentProps<typeof ChartTooltipContent> & {
  grouping: string;
  isShare: boolean;
  preparedByPackage: ReadonlyMap<string, ReadonlyMap<string, PreparedDownload>>;
};

export const DownloadTooltip = ({
  grouping,
  isShare,
  preparedByPackage,
  ...props
}: DownloadTooltipProps) => {
  const date = props.payload?.[0]?.payload?.date;
  const points = [...preparedByPackage.values()].map((series) =>
    series.get(date)
  );
  const hasEstimates = points.some((point) => (point?.estimatedDays ?? 0) > 0);
  const reportedSummary = [...preparedByPackage.entries()]
    .flatMap(([name, series]) => {
      const point = series.get(date);
      return point && (isShare || point.estimatedDays > 0)
        ? [`${name}: ${point.reportedDownloads.toLocaleString("en-US")}`]
        : [];
    })
    .join(" · ");
  let estimateNote = "≈ Includes estimated days";
  if (isShare) {
    estimateNote = "≈ Shares include estimated downloads";
  } else if (grouping === "day") {
    estimateNote = "≈ Estimated · npm reported 0";
  }
  const formatDate = (value: string) =>
    new Date(`${value}T00:00:00Z`).toLocaleDateString("en-US", {
      month: "short",
      day: "numeric",
      timeZone: "UTC",
    });

  return (
    <ChartTooltipContent
      {...props}
      footer={
        hasEstimates ? (
          <div className="max-w-64 pt-1 text-muted-foreground text-xs">
            <p>{estimateNote}</p>
            {(isShare || grouping !== "day") && (
              <p className="pt-1">npm reported: {reportedSummary}</p>
            )}
          </div>
        ) : undefined
      }
      formatter={(value, name, item) => {
        const point = preparedByPackage.get(String(name))?.get(date);
        const estimated = isShare
          ? hasEstimates
          : (point?.estimatedDays ?? 0) > 0;
        const formatted = isShare
          ? `${Number(value).toFixed(1)}%`
          : Number(value).toLocaleString("en-US");
        const reported = point?.reportedDownloads.toLocaleString("en-US");
        return (
          <div className="flex w-full items-center justify-between gap-6">
            <span className="flex items-center gap-2 text-muted-foreground">
              <span
                aria-hidden="true"
                className="size-2 shrink-0 rounded-[2px]"
                style={{ backgroundColor: item.color }}
              />
              {name}
            </span>
            <span
              className="flex items-baseline gap-1 font-medium font-mono tabular-nums"
              title={
                reported === undefined
                  ? undefined
                  : `npm reported: ${reported} downloads`
              }
            >
              {estimated && (
                <span aria-hidden="true" className="text-muted-foreground">
                  ≈
                </span>
              )}
              {formatted}
              {estimated && (
                <span className="sr-only">
                  {" "}
                  Estimated; npm reported {reported} downloads.
                </span>
              )}
            </span>
          </div>
        );
      }}
      indicator="dot"
      labelFormatter={() => {
        if (!date) {
          return props.label;
        }
        const end = props.payload?.[0]?.payload?.dateEnd;
        return grouping === "day" || !end
          ? formatDate(date)
          : `${formatDate(date)} - ${formatDate(end)}`;
      }}
    />
  );
};
````

## File: components/embed-dialog.tsx
````typescript
"use client";

import { CheckIcon, CodeIcon, CopyIcon } from "lucide-react";
import { useCallback, useMemo, useState } from "react";
import {
  useGrouping,
  useMetric,
  usePackages,
  useTimeRange,
  useZeroMode,
} from "@/providers/filters";
import { Button } from "./ui/button";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "./ui/dialog";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "./ui/tabs";

type SnippetType = "markdown" | "html" | "url";

export const EmbedDialog = () => {
  const [tab, setTab] = useState<SnippetType>("markdown");
  const [copied, setCopied] = useState(false);

  const [packages] = usePackages();
  const [timeRange] = useTimeRange();
  const [grouping] = useGrouping();
  const [metric] = useMetric();
  const [zeroMode] = useZeroMode();

  const handleTabChange = useCallback((value: string) => {
    setTab(value as SnippetType);
  }, []);

  const svgUrl = useMemo(() => {
    const baseUrl =
      typeof window === "undefined" ? "/svg" : `${window.location.origin}/svg`;

    const params = new URLSearchParams();

    if (packages.length > 0) {
      params.set("q", packages.join(","));
    }

    params.set("timeRange", timeRange);
    params.set("grouping", grouping);
    params.set("metric", metric);
    params.set("zeroMode", zeroMode);

    return `${baseUrl}?${params.toString()}`;
  }, [packages, timeRange, grouping, metric, zeroMode]);

  const snippets = useMemo(
    () => ({
      markdown: `![npm downloads](${svgUrl})`,
      html: `<img src="${svgUrl}" alt="npm downloads" />`,
      url: svgUrl,
    }),
    [svgUrl]
  );

  const handleCopyToClipboard = useCallback(async () => {
    try {
      await navigator.clipboard.writeText(snippets[tab]);
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch (err) {
      console.error("Failed to copy:", err);
    }
  }, [tab, snippets]);

  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button disabled={packages.length === 0} size="icon" variant="outline">
          <CodeIcon />
          <span className="sr-only">Embed</span>
        </Button>
      </DialogTrigger>

      <DialogContent className="sm:max-w-3xl">
        <DialogHeader>
          <DialogTitle>Embed Chart</DialogTitle>
          <DialogDescription>
            Copy the code below to embed this chart in your documentation or
            website.
          </DialogDescription>
        </DialogHeader>

        <div className="relative overflow-auto">
          <Button
            className="absolute top-1 right-0"
            onClick={handleCopyToClipboard}
            size="icon-sm"
            variant="secondary"
          >
            {copied ? <CheckIcon /> : <CopyIcon />}
          </Button>

          <Tabs onValueChange={handleTabChange} value={tab}>
            <TabsList>
              <TabsTrigger value="markdown">Markdown</TabsTrigger>
              <TabsTrigger value="html">HTML</TabsTrigger>
              <TabsTrigger value="url">URL</TabsTrigger>
            </TabsList>

            {(Object.keys(snippets) as SnippetType[]).map((type) => (
              <TabsContent key={type} value={type}>
                <button
                  className="w-full cursor-copy text-left"
                  onClick={handleCopyToClipboard}
                  type="button"
                >
                  <pre className="overflow-x-auto rounded-lg border p-4 text-sm">
                    <code>{snippets[type]}</code>
                  </pre>
                </button>
              </TabsContent>
            ))}
          </Tabs>

          <div className="mt-4">
            {/* biome-ignore lint/performance/noImgElement: Dynamic SVG from API endpoint */}
            {/* biome-ignore lint/correctness/useImageSize: SVG scales responsively */}
            <img alt="Chart preview" className="h-auto w-full" src={svgUrl} />
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
};
````

## File: components/empty-state.tsx
````typescript
import { LineChartIcon } from "lucide-react";
import {
  Empty,
  EmptyContent,
  EmptyDescription,
  EmptyHeader,
  EmptyMedia,
  EmptyTitle,
} from "@/components/ui/empty";
import { usePackages } from "@/providers/filters";
import { Badge } from "./ui/badge";

const examples = [
  {
    label: "Next.js vs Nuxt",
    packages: ["next", "nuxt"],
  },
  {
    label: "React vs Angular",
    packages: ["react", "@angular/core"],
  },
  {
    label: "Express vs Hono",
    packages: ["express", "hono"],
  },
  {
    label: "Tailwind CSS vs Bootstrap",
    packages: ["tailwindcss", "bootstrap"],
  },
  {
    label: "ESLint vs Biome vs Oxlint",
    packages: ["eslint", "@biomejs/biome", "oxlint"],
  },
  {
    label: "Vite vs Webpack",
    packages: ["vite", "webpack"],
  },
  {
    label: "Jest vs Mocha vs Vitest",
    packages: ["jest", "mocha", "vitest"],
  },
  {
    label: "Prisma vs Drizzle",
    packages: ["prisma", "drizzle-orm"],
  },
  {
    label: "Radix UI vs Base UI",
    packages: ["radix-ui", "@base-ui-components/react"],
  },
  {
    label: "Svelte vs Solid vs Remix",
    packages: ["svelte", "solid-js", "@remix-run/router"],
  },
  {
    label: "Zod vs Valibot",
    packages: ["zod", "valibot"],
  },
];

export const EmptyState = () => {
  const [_packages, setPackages] = usePackages();

  return (
    <Empty className="size-full rounded-lg border border-solid bg-background">
      <EmptyHeader>
        <EmptyMedia variant="icon">
          <LineChartIcon />
        </EmptyMedia>
        <EmptyTitle>No packages selected</EmptyTitle>
        <EmptyDescription>
          You haven&apos;t selected any packages yet. Get started by selecting a
          package or try one of the examples below.
        </EmptyDescription>
      </EmptyHeader>
      <EmptyContent className="max-w-lg">
        <div className="flex flex-wrap items-center justify-center gap-2">
          {examples.map((example) => (
            <button
              key={example.label}
              onClick={() => setPackages(example.packages)}
              type="button"
            >
              <Badge
                className="cursor-pointer hover:bg-sidebar"
                key={example.label}
                variant="secondary"
              >
                {example.label}
              </Badge>
            </button>
          ))}
        </div>
      </EmptyContent>
    </Empty>
  );
};
````

## File: components/footer.tsx
````typescript
"use client";

import { usePackages } from "@/providers/filters";
import { PackageSearch } from "./package-search";

export const Footer = () => {
  const [packages, setPackages] = usePackages();

  return (
    <footer className="flex items-center justify-center p-4">
      <PackageSearch
        onPackagesChange={(nextPackages) => setPackages(nextPackages)}
        packages={packages}
      />
    </footer>
  );
};
````

## File: components/github.tsx
````typescript
import { Button } from "./ui/button";

export const GitHub = () => (
  <Button asChild className="shadow-none" size="icon" variant="outline">
    <a
      href="https://github.com/vercel/npm.bet"
      rel="noopener noreferrer"
      target="_blank"
    >
      <svg viewBox="0 0 98 96" xmlns="http://www.w3.org/2000/svg">
        <title>GitHub</title>
        <path
          clipRule="evenodd"
          d="M48.854 0C21.839 0 0 22 0 49.217c0 21.756 13.993 40.172 33.405 46.69 2.427.49 3.316-1.059 3.316-2.362 0-1.141-.08-5.052-.08-9.127-13.59 2.934-16.42-5.867-16.42-5.867-2.184-5.704-5.42-7.17-5.42-7.17-4.448-3.015.324-3.015.324-3.015 4.934.326 7.523 5.052 7.523 5.052 4.367 7.496 11.404 5.378 14.235 4.074.404-3.178 1.699-5.378 3.074-6.6-10.839-1.141-22.243-5.378-22.243-24.283 0-5.378 1.94-9.778 5.014-13.2-.485-1.222-2.184-6.275.486-13.038 0 0 4.125-1.304 13.426 5.052a46.97 46.97 0 0 1 12.214-1.63c4.125 0 8.33.571 12.213 1.63 9.302-6.356 13.427-5.052 13.427-5.052 2.67 6.763.97 11.816.485 13.038 3.155 3.422 5.015 7.822 5.015 13.2 0 18.905-11.404 23.06-22.324 24.283 1.78 1.548 3.316 4.481 3.316 9.126 0 6.6-.08 11.897-.08 13.526 0 1.304.89 2.853 3.316 2.364 19.412-6.52 33.405-24.935 33.405-46.691C97.707 22 75.788 0 48.854 0z"
          fill="currentColor"
          fillRule="evenodd"
        />
      </svg>
      <span className="sr-only">GitHub</span>
    </a>
  </Button>
);
````

## File: components/grouping-selector.tsx
````typescript
"use client";

import { GroupIcon } from "lucide-react";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { cn } from "@/lib/utils";
import { useGrouping } from "@/providers/filters";

interface GroupingSelectorProps {
  className?: string;
}

export const GroupingSelector = ({ className }: GroupingSelectorProps) => {
  const [grouping, setGrouping] = useGrouping();

  return (
    <Select onValueChange={setGrouping} value={grouping}>
      <SelectTrigger
        className={cn(
          "w-36 bg-background shadow-none [&>span]:flex-1",
          className
        )}
      >
        <GroupIcon className="size-4" />
        <SelectValue placeholder="Select grouping" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="day">By day</SelectItem>
        <SelectItem value="week">By week</SelectItem>
        <SelectItem value="month">By month</SelectItem>
      </SelectContent>
    </Select>
  );
};
````

## File: components/header.tsx
````typescript
"use client";

import { SettingsIcon } from "lucide-react";
import useSWR from "swr";
import { getPackageData, type PackageData } from "@/actions/package/get";
import {
  Drawer,
  DrawerClose,
  DrawerContent,
  DrawerDescription,
  DrawerFooter,
  DrawerHeader,
  DrawerTitle,
  DrawerTrigger,
} from "@/components/ui/drawer";
import { usePackages, useTimeRange } from "@/providers/filters";
import { EmbedDialog } from "./embed-dialog";
import { GitHub } from "./github";
import { GroupingSelector } from "./grouping-selector";
import { Logo } from "./logo";
import { MetricSelector } from "./metric-selector";
import { Screenshot } from "./screenshot";
import { ThemeToggle } from "./theme-toggle";
import { TimeRangeSelector } from "./time-range-selector";
import { Button } from "./ui/button";
import { ButtonGroup } from "./ui/button-group";
import { ZeroModeToggle } from "./zero-mode-toggle";

export const Header = () => {
  const [packages] = usePackages();
  const [timeRange] = useTimeRange();

  const { data } = useSWR<PackageData[]>(
    packages.length > 0 ? [packages, timeRange] : null,
    async ([pkgs, range]: [string[], string]) =>
      Promise.all(pkgs.map((pkg) => getPackageData(pkg, range)))
  );

  return (
    <header className="flex items-center justify-between">
      <Logo />
      <div className="flex items-center gap-2">
        <TimeRangeSelector className="hidden sm:flex" />
        <GroupingSelector className="hidden sm:flex" />
        {packages.length > 1 && <MetricSelector className="hidden sm:flex" />}
        <ZeroModeToggle className="hidden sm:inline-flex" />

        <ButtonGroup className="sm:hidden">
          <Drawer>
            <DrawerTrigger asChild>
              <Button className="shadow-none" size="icon" variant="outline">
                <SettingsIcon />
              </Button>
            </DrawerTrigger>
            <DrawerContent>
              <div className="mx-auto w-full max-w-sm">
                <DrawerHeader>
                  <DrawerTitle>Settings</DrawerTitle>
                  <DrawerDescription>Customize your chart.</DrawerDescription>
                </DrawerHeader>
                <div className="flex flex-col gap-2 p-4 pb-0">
                  <TimeRangeSelector className="w-full" />
                  <GroupingSelector className="w-full" />
                  <div className="flex items-center gap-2">
                    {packages.length > 1 && <MetricSelector />}
                    <ZeroModeToggle showLabel size="sm" />
                  </div>
                </div>
                <DrawerFooter>
                  <DrawerClose asChild>
                    <Button variant="outline">Close</Button>
                  </DrawerClose>
                </DrawerFooter>
              </div>
            </DrawerContent>
          </Drawer>
          <ThemeToggle />
        </ButtonGroup>

        <ButtonGroup className="hidden sm:flex">
          <Screenshot data={data} />
          <EmbedDialog />
          <ThemeToggle />
        </ButtonGroup>

        <GitHub />
      </div>
    </header>
  );
};
````

## File: components/logo.tsx
````typescript
import Link from "next/link";
import type { ComponentProps } from "react";
import { Button } from "./ui/button";

const VercelLogo = (props: ComponentProps<"svg">) => (
  <svg
    {...props}
    fill="none"
    viewBox="0 0 76 65"
    xmlns="http://www.w3.org/2000/svg"
  >
    <title>Vercel</title>
    <path d="M37.5274 0L75.0548 65H0L37.5274 0Z" fill="currentColor" />
  </svg>
);

export const Logo = () => (
  <Button asChild size="sm" variant="ghost">
    <Link className="flex items-center gap-2" href="/">
      <VercelLogo className="size-4" />
      <span className="pointer-events-none select-none text-muted-foreground opacity-50">
        /
      </span>
      <p>npm.bet</p>
    </Link>
  </Button>
);
````

## File: components/main.tsx
````typescript
"use client";

import { useEffect } from "react";
import useSWR from "swr";
import { getPackageData, type PackageData } from "@/actions/package/get";
import {
  useGrouping,
  useMetric,
  usePackages,
  useTimeRange,
} from "@/providers/filters";
import { ChartAreaInteractive } from "./chart";
import { ChartLoading } from "./chart-loading";
import { EmptyState } from "./empty-state";
import { Alert, AlertDescription, AlertTitle } from "./ui/alert";

export const Main = () => {
  const [timeRange] = useTimeRange();
  const [packages] = usePackages();
  const [grouping] = useGrouping();
  const [metric] = useMetric();

  const { data: packageData, error } = useSWR<PackageData[]>(
    packages.length > 0 ? [packages, timeRange] : null,
    async ([pkgs, range]: [string[], string]) =>
      Promise.all(pkgs.map((pkg) => getPackageData(pkg, range)))
  );

  useEffect(() => {
    if (typeof window === "undefined") {
      return;
    }

    if (packages.length > 0) {
      document.title = `${packages.join(" vs ")} | npm.bet`;
      document
        .querySelector("meta[property='og:image']")
        ?.setAttribute(
          "content",
          `${window.location.origin}/og?q=${packages.join(",")}&timeRange=${timeRange}&grouping=${grouping}&metric=${metric}`
        );
    } else {
      document.title = "npm.bet";
      document
        .querySelector("meta[property='og:image']")
        ?.setAttribute("content", `${window.location.origin}/og`);
    }
  }, [packages, timeRange, grouping, metric]);

  if (packages.length === 0) {
    return (
      <main className="overflow-hidden">
        <EmptyState />
      </main>
    );
  }

  if (error) {
    return (
      <main className="flex size-full items-center justify-center overflow-hidden">
        <Alert
          className="max-w-md border-destructive bg-destructive/5"
          variant="destructive"
        >
          <AlertTitle>Error loading package data</AlertTitle>
          <AlertDescription>Please try again later.</AlertDescription>
        </Alert>
      </main>
    );
  }

  if (!packageData) {
    return (
      <main className="overflow-hidden">
        <ChartLoading />
      </main>
    );
  }

  return (
    <main className="overflow-hidden">
      <ChartAreaInteractive data={packageData} />
    </main>
  );
};
````

## File: components/metric-selector.tsx
````typescript
"use client";

import { ToggleGroup, ToggleGroupItem } from "@/components/ui/toggle-group";
import { cn } from "@/lib/utils";
import { useMetric } from "@/providers/filters";

interface MetricSelectorProps {
  className?: string;
}

export const MetricSelector = ({ className }: MetricSelectorProps) => {
  const [metric, setMetric] = useMetric();

  return (
    <ToggleGroup
      className={cn("bg-background", className)}
      onValueChange={(value) => {
        if (value) {
          setMetric(value);
        }
      }}
      size="sm"
      type="single"
      value={metric}
      variant="outline"
    >
      <ToggleGroupItem value="downloads">Downloads</ToggleGroupItem>
      <ToggleGroupItem value="share">Share %</ToggleGroupItem>
    </ToggleGroup>
  );
};
````

## File: components/package-search.tsx
````typescript
"use client";

import { Command as CommandPrimitive } from "cmdk";
import { PackageSearchIcon, XIcon } from "lucide-react";
import { useEffect, useState } from "react";
import useSWR from "swr";
import {
  type NpmSearchResponse,
  searchPackages,
} from "@/actions/package/search";
import { cn } from "@/lib/utils";
import { Badge } from "./ui/badge";
import {
  Command,
  CommandEmpty,
  CommandGroup,
  CommandItem,
  CommandList,
} from "./ui/command";

const DEBOUNCE_MS = 200;

interface PackageSearchProps {
  onPackagesChange: (packages: string[]) => void;
  packages: string[];
}

export const PackageSearch = ({
  onPackagesChange,
  packages,
}: PackageSearchProps) => {
  const [value, setValue] = useState("");
  const [debouncedValue, setDebouncedValue] = useState("");

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, DEBOUNCE_MS);

    return () => {
      clearTimeout(timer);
    };
  }, [value]);

  const { data, error } = useSWR<NpmSearchResponse>(
    debouncedValue ? debouncedValue.toLowerCase() : null,
    searchPackages,
    {
      keepPreviousData: true,
      revalidateOnFocus: false,
    }
  );

  const handleSelect = (packageName: string) => {
    if (!packages.includes(packageName)) {
      onPackagesChange([...packages, packageName]);
    }
    setValue("");
  };

  const handleRemove = (packageName: string) => {
    onPackagesChange(packages.filter((pkg) => pkg !== packageName));
  };

  const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === "Enter" && value.trim() !== "") {
      event.preventDefault();
      handleSelect(value.trim());
    } else if (
      event.key === "Backspace" &&
      value === "" &&
      packages.length > 0
    ) {
      event.preventDefault();
      const newPackages = [...packages];
      newPackages.pop();
      onPackagesChange(newPackages);
    }
  };

  const shouldShowResults = debouncedValue.trim().length > 0;
  const showEmptyState = data && data.objects.length === 0;
  const showResults = data && data.objects.length > 0;

  return (
    <div className="absolute bottom-4 left-1/2 w-full max-w-xs -translate-x-1/2 md:max-w-md">
      <Command className="w-full rounded-lg border">
        <CommandList>
          {shouldShowResults && error ? (
            <CommandEmpty>Failed to load packages. Try again.</CommandEmpty>
          ) : null}
          {shouldShowResults && showEmptyState ? (
            <CommandEmpty className="flex items-center gap-2 p-4 text-muted-foreground text-sm">
              <PackageSearchIcon className="size-3" /> No packages found.
            </CommandEmpty>
          ) : null}
          {shouldShowResults && showResults ? (
            <CommandGroup>
              {data.objects.map((item) => (
                <CommandItem
                  key={item.package.name}
                  onSelect={() => handleSelect(item.package.name)}
                  value={item.package.name}
                >
                  <span className="shrink-0 truncate font-medium">
                    {item.package.name}
                  </span>
                  <span className="truncate text-muted-foreground text-xs">
                    {item.package.description}
                  </span>
                </CommandItem>
              ))}
            </CommandGroup>
          ) : null}
        </CommandList>
        <div className="flex items-center gap-2 overflow-x-auto px-3">
          <PackageSearchIcon className="size-4 shrink-0 opacity-50" />
          {packages.map((pkg) => (
            <Badge className="gap-1 pr-1 pl-2" key={pkg} variant="secondary">
              {pkg}
              <button
                aria-label={`Remove ${pkg}`}
                className="rounded-sm p-0.5 transition-colors hover:bg-secondary-foreground/20"
                onClick={() => handleRemove(pkg)}
                type="button"
              >
                <XIcon className="size-3" />
              </button>
            </Badge>
          ))}
          <CommandPrimitive.Input
            autoCapitalize="off"
            autoFocus
            className={cn(
              "flex flex-1 rounded-md bg-transparent py-2 text-sm outline-hidden placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
            )}
            onKeyDown={handleKeyDown}
            onValueChange={setValue}
            placeholder="Search..."
            value={value}
          />
        </div>
      </Command>
    </div>
  );
};
````

## File: components/screenshot.tsx
````typescript
"use client";

import { toPng } from "html-to-image";
import { CheckIcon, CopyIcon as CopyIconRaw, DownloadIcon } from "lucide-react";
import { useRef, useState } from "react";
import { toast } from "sonner";
import type { PackageData } from "@/actions/package/get";
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { cn } from "@/lib/utils";
import { ChartAreaInteractive } from "./chart";
import { Logo } from "./logo";

interface ScreenshotProps {
  data?: PackageData[];
}

const colors = [
  "var(--color-secondary)",
  "var(--color-chart-1)",
  "var(--color-chart-2)",
  "var(--color-chart-3)",
  "var(--color-chart-4)",
  "var(--color-chart-5)",
];

export const Screenshot = ({ data }: ScreenshotProps) => {
  const [isOpen, setIsOpen] = useState(false);
  const [isDownloading, setIsDownloading] = useState(false);
  const chartRef = useRef<HTMLDivElement>(null);
  const [backgroundColor, setBackgroundColor] = useState(colors[0]);
  const [isCopied, setIsCopied] = useState(false);

  const handleCopy = async () => {
    if (!chartRef.current) {
      return;
    }

    try {
      const dataUrl = await toPng(chartRef.current, {
        cacheBust: true,
        pixelRatio: 2,
      });
      const blob = await fetch(dataUrl).then((res) => res.blob());
      const clipboardItem = new ClipboardItem({
        "image/png": blob,
      });

      await navigator.clipboard.write([clipboardItem]);

      setIsCopied(true);

      setTimeout(() => {
        setIsCopied(false);
      }, 2000);
    } catch (error) {
      const message =
        error instanceof Error ? error.message : "Failed to copy to clipboard";
      setIsCopied(false);
      toast.error(message);
    }
  };

  const CopyIcon = isCopied ? CheckIcon : CopyIconRaw;

  const handleDownload = async () => {
    if (!chartRef.current) {
      return;
    }

    try {
      setIsDownloading(true);

      const dataUrl = await toPng(chartRef.current, {
        cacheBust: true,
        pixelRatio: 2,
      });

      const link = document.createElement("a");
      link.download = `npm-bet-chart-${Date.now()}.png`;
      link.href = dataUrl;
      link.click();
    } catch (error) {
      console.error("Failed to generate screenshot:", error);
    } finally {
      setIsDownloading(false);
    }
  };

  return (
    <Dialog onOpenChange={setIsOpen} open={isOpen}>
      <DialogTrigger asChild>
        <Button
          className="shadow-none"
          disabled={!data || data.length === 0}
          size="icon"
          variant="outline"
        >
          <DownloadIcon />
          <span className="sr-only">Download chart screenshot</span>
        </Button>
      </DialogTrigger>
      <DialogContent className="w-full sm:max-w-4xl">
        <DialogHeader>
          <DialogTitle>Download Chart</DialogTitle>
        </DialogHeader>
        <div className="space-y-4">
          <div
            className="rounded-lg p-12 pb-8"
            ref={chartRef}
            style={{ backgroundColor }}
          >
            <ChartAreaInteractive data={data ?? []} />
            <div
              className={cn(
                "mt-8 flex items-center justify-center",
                backgroundColor === "var(--color-secondary)"
                  ? "text-foreground"
                  : "text-white"
              )}
            >
              <Logo />
            </div>
          </div>
          <div className="flex justify-between">
            <div className="flex items-center gap-2">
              {colors.map((color) => (
                <Button
                  className={cn(
                    "size-6 overflow-hidden rounded-full p-0 shadow-none",
                    color === backgroundColor && "ring-2 ring-ring"
                  )}
                  key={color}
                  onClick={() => setBackgroundColor(color)}
                  size="icon-sm"
                  variant="outline"
                >
                  <div
                    className="size-full"
                    style={{ backgroundColor: color }}
                  />
                </Button>
              ))}
            </div>
            <div className="flex items-center gap-2">
              <Button
                disabled={isCopied}
                onClick={handleCopy}
                variant="outline"
              >
                <CopyIcon />
                {isCopied ? "Copied!" : "Copy to clipboard"}
              </Button>
              <Button
                disabled={isDownloading}
                onClick={handleDownload}
                variant="default"
              >
                <DownloadIcon />
                {isDownloading ? "Downloading..." : "Download PNG"}
              </Button>
            </div>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
};
````

## File: components/theme-toggle.tsx
````typescript
import { MoonIcon, SunIcon } from "lucide-react";
import { useTheme } from "next-themes";
import { Button } from "./ui/button";

export const ThemeToggle = () => {
  const { theme, setTheme } = useTheme();

  const Icon = theme === "dark" ? SunIcon : MoonIcon;

  return (
    <Button
      className="shadow-none"
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
      size="icon"
      variant="outline"
    >
      <Icon className="size-4" />
    </Button>
  );
};
````

## File: components/time-range-selector.tsx
````typescript
"use client";

import { CalendarIcon } from "lucide-react";
import { useMemo } from "react";
import type { DateRange } from "react-day-picker";
import { cn } from "@/lib/utils";
import { useTimeRange } from "@/providers/filters";
import { Button } from "./ui/button";
import { Calendar } from "./ui/calendar";
import { Card, CardContent } from "./ui/card";
import { Popover, PopoverContent, PopoverTrigger } from "./ui/popover";

const PREDEFINED_RANGES = [
  { label: "Last week", value: "last-week" },
  { label: "Last month", value: "last-month" },
  { label: "Last year", value: "last-year" },
  { label: "Last 2 years", value: "last-2-years" },
  { label: "Last 5 years", value: "last-5-years" },
  { label: "All time", value: "all-time" },
] as const;

const formatDateForAPI = (date: Date): string => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");

  return `${year}-${month}-${day}`;
};

const formatDateForDisplay = (date: Date): string =>
  date.toLocaleDateString("en-US", {
    month: "short",
    day: "numeric",
    year: "numeric",
  });

const parseDateRangeFromString = (
  rangeString: string
): DateRange | undefined => {
  if (!rangeString.includes(":")) {
    return;
  }

  const [fromStr, toStr] = rangeString.split(":");

  // Parse dates with explicit time to avoid timezone issues
  // YYYY-MM-DD format when parsed creates UTC midnight, but we want local midnight
  const [fromYear, fromMonth, fromDay] = fromStr.split("-").map(Number);
  const [toYear, toMonth, toDay] = toStr.split("-").map(Number);

  const from = new Date(fromYear, fromMonth - 1, fromDay);
  const to = new Date(toYear, toMonth - 1, toDay);

  if (Number.isNaN(from.getTime()) || Number.isNaN(to.getTime())) {
    return;
  }

  return { from, to };
};

interface TimeRangeSelectorProps {
  className?: string;
}

export const TimeRangeSelector = ({ className }: TimeRangeSelectorProps) => {
  const [timeRange, setTimeRange] = useTimeRange();

  const handleRangeSelect = (range: DateRange | undefined) => {
    if (range?.from && range?.to) {
      const formattedRange = `${formatDateForAPI(range.from)}:${formatDateForAPI(range.to)}`;
      setTimeRange(formattedRange);
    }
  };

  const selectedRange = PREDEFINED_RANGES.find(
    (range) => range.value === timeRange
  );

  const customDateRange = useMemo(() => {
    if (selectedRange) {
      return;
    }
    return parseDateRangeFromString(timeRange);
  }, [timeRange, selectedRange]);

  const displayText = useMemo(() => {
    if (selectedRange) {
      return selectedRange.label;
    }

    if (customDateRange?.from && customDateRange?.to) {
      return `${formatDateForDisplay(customDateRange.from)} - ${formatDateForDisplay(customDateRange.to)}`;
    }

    return "Select range";
  }, [selectedRange, customDateRange]);

  return (
    <Popover>
      <PopoverTrigger asChild>
        <Button
          className={cn("w-64 text-left font-normal shadow-none", className)}
          variant="outline"
        >
          <CalendarIcon className="size-4 text-muted-foreground" />
          <span className="flex-1 truncate text-sm">{displayText}</span>
        </Button>
      </PopoverTrigger>
      <PopoverContent align="end" className="w-auto p-0">
        <Card className="gap-0 border-0 p-0 shadow-none">
          <CardContent className="relative flex flex-col p-0 md:flex-row">
            <div className="border-b p-6 md:border-r md:border-b-0">
              <Calendar
                className="bg-transparent p-0"
                formatters={{
                  formatWeekdayName: (date) =>
                    date.toLocaleString("en-US", { weekday: "short" }),
                }}
                mode="range"
                onSelect={handleRangeSelect}
                selected={customDateRange}
                showOutsideDays={false}
              />
            </div>
            <div className="flex w-full flex-col gap-2 p-6 md:w-48">
              <div className="mb-2 font-medium text-sm">Predefined Ranges</div>
              {PREDEFINED_RANGES.map((range) => (
                <Button
                  className="w-full justify-start shadow-none"
                  key={range.value}
                  onClick={() => setTimeRange(range.value)}
                  variant={timeRange === range.value ? "default" : "outline"}
                >
                  {range.label}
                </Button>
              ))}
            </div>
          </CardContent>
        </Card>
      </PopoverContent>
    </Popover>
  );
};
````

## File: components/zero-mode-toggle.tsx
````typescript
"use client";

import { SplineIcon } from "lucide-react";
import { useId } from "react";
import { Toggle } from "@/components/ui/toggle";
import {
  Tooltip,
  TooltipContent,
  TooltipTrigger,
} from "@/components/ui/tooltip";
import { cn } from "@/lib/utils";
import { useZeroMode } from "@/providers/filters";

export const ZeroModeToggle = ({
  className,
  size = "default",
  showLabel = false,
}: {
  className?: string;
  size?: "default" | "sm";
  showLabel?: boolean;
}) => {
  const id = useId();
  const [zeroMode, setZeroMode] = useZeroMode();

  return (
    <Tooltip>
      <TooltipTrigger asChild>
        <span className={cn("inline-flex shrink-0", className)}>
          <Toggle
            aria-describedby={id}
            aria-label="Interpolate zeros"
            onPressedChange={(pressed) =>
              setZeroMode(pressed ? "estimated" : "reported")
            }
            pressed={zeroMode === "estimated"}
            size={size}
            variant="outline"
          >
            {showLabel ? "Interpolate" : <SplineIcon />}
          </Toggle>
        </span>
      </TooltipTrigger>
      <TooltipContent sideOffset={6}>Interpolate zeros</TooltipContent>
      <span className="sr-only" id={id}>
        Estimate one- or two-day zero-download gaps between reported positive
        days. Estimates are not confirmed download counts.
      </span>
    </Tooltip>
  );
};
````

## File: hooks/use-mobile.ts
````typescript
import * as React from "react"

const MOBILE_BREAKPOINT = 768

export function useIsMobile() {
  const [isMobile, setIsMobile] = React.useState<boolean | undefined>(undefined)

  React.useEffect(() => {
    const mql = window.matchMedia(`(max-width: ${MOBILE_BREAKPOINT - 1}px)`)
    const onChange = () => {
      setIsMobile(window.innerWidth < MOBILE_BREAKPOINT)
    }
    mql.addEventListener("change", onChange)
    setIsMobile(window.innerWidth < MOBILE_BREAKPOINT)
    return () => mql.removeEventListener("change", onChange)
  }, [])

  return !!isMobile
}
````

## File: lib/chart-utils.ts
````typescript
/**
 * Get the start date of the week for a given date
 * Week starts on Sunday (weekStartsOn: 0)
 */
export const getWeekStart = (date: Date): string => {
  const weekStart = new Date(
    Date.UTC(
      date.getUTCFullYear(),
      date.getUTCMonth(),
      date.getUTCDate() - date.getUTCDay()
    )
  );
  return weekStart.toISOString().split("T")[0];
};

/**
 * Get the start date of the month for a given date
 */
export const getMonthStart = (date: Date): string => {
  const monthStart = new Date(
    Date.UTC(date.getUTCFullYear(), date.getUTCMonth(), 1)
  );
  return monthStart.toISOString().split("T")[0];
};

/**
 * Group download data by day, week, or month
 */
export const groupData = (
  downloads: { downloads: number; day: string }[],
  groupBy: string
): { date: string; downloads: number }[] => {
  if (groupBy === "day") {
    return downloads.map((item) => ({
      date: item.day,
      downloads: item.downloads,
    }));
  }

  const getGroupKey = (date: Date): string => {
    if (groupBy === "week") {
      return getWeekStart(date);
    }
    return getMonthStart(date);
  };

  const grouped = downloads.reduce(
    (acc, item) => {
      const groupKey = getGroupKey(new Date(item.day));
      if (!acc[groupKey]) {
        acc[groupKey] = 0;
      }
      acc[groupKey] += item.downloads;
      return acc;
    },
    {} as Record<string, number>
  );

  return Object.entries(grouped).map(([date, downloadCount]) => ({
    date,
    downloads: downloadCount,
  }));
};

/**
 * Check if a date should be removed because it's an incomplete period
 */
export const shouldRemoveIncompleteDate = (
  dateString: string,
  grouping: string
): boolean => {
  const date = new Date(`${dateString}T00:00:00.000Z`);
  const now = new Date();

  if (grouping === "day") {
    // For daily grouping, check if it's today
    return dateString === now.toISOString().slice(0, 10);
  }
  if (grouping === "week") {
    // For weekly grouping, check if we're in the same week
    return getWeekStart(date) === getWeekStart(now);
  }
  if (grouping === "month") {
    // For monthly grouping, check if we're in the same month
    return getMonthStart(date) === getMonthStart(now);
  }

  return false;
};

/**
 * Escape a string for safe interpolation into XML/SVG
 */
export const escapeXml = (str: string): string =>
  str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&apos;");

/**
 * Format a number for display (e.g., 1000 -> 1K, 1000000 -> 1.0M)
 */
export const formatNumber = (value: number): string => {
  if (value >= 1_000_000) {
    return `${(value / 1_000_000).toFixed(1)}M`;
  }
  if (value >= 1000) {
    return `${(value / 1000).toFixed(0)}K`;
  }
  return value.toString();
};
````

## File: lib/download-data.ts
````typescript
import { groupData } from "./chart-utils";

export const zeroModes = ["reported", "estimated"] as const;
export type ZeroMode = (typeof zeroModes)[number];

export interface DailyDownload {
  day: string;
  downloads: number;
}

export interface PreparedDownload {
  date: string;
  downloads: number;
  reportedDownloads: number;
  estimatedDays: number;
}

export interface InterpolatedDownload extends DailyDownload {
  reportedDownloads: number;
  estimated: boolean;
}

const DAY_MS = 86_400_000;
const MAX_ZERO_DAYS = 7;

const WEEKDAY_WINDOW = 3;

// Mean of positive reported counts on the same weekday in nearby weeks.
const weekdayBaseline = (
  downloads: readonly DailyDownload[],
  dates: readonly number[],
  index: number
): number | undefined => {
  let sum = 0;
  let count = 0;
  for (let week = -WEEKDAY_WINDOW; week <= WEEKDAY_WINDOW; week++) {
    const other = index + week * 7;
    const value = downloads[other]?.downloads;
    if (
      week !== 0 &&
      value > 0 &&
      dates[other] - dates[index] === week * 7 * DAY_MS
    ) {
      sum += value;
      count++;
    }
  }
  return count > 0 ? sum / count : undefined;
};

export const parseZeroMode = (value: unknown): ZeroMode =>
  value === "estimated" ? "estimated" : "reported";

export const interpolateZeroDownloads = (
  downloads: readonly DailyDownload[]
): InterpolatedDownload[] => {
  let previousTime = Number.NEGATIVE_INFINITY;
  const dates = downloads.map(({ day, downloads: count }) => {
    const time = Date.parse(`${day}T00:00:00.000Z`);
    if (
      !Number.isFinite(time) ||
      new Date(time).toISOString().slice(0, 10) !== day ||
      time <= previousTime ||
      !Number.isSafeInteger(count) ||
      count < 0
    ) {
      throw new RangeError(
        "Expected ordered unique UTC days and nonnegative integer downloads"
      );
    }
    previousTime = time;
    return time;
  });
  const result = downloads.map((item) => ({
    ...item,
    reportedDownloads: item.downloads,
    estimated: false,
  }));

  for (let start = 0; start < downloads.length; start++) {
    if (downloads[start].downloads !== 0) {
      continue;
    }
    let end = start;
    while (end < downloads.length && downloads[end].downloads === 0) {
      end++;
    }
    const length = end - start;
    const left = downloads[start - 1];
    const right = downloads[end];
    if (
      length <= MAX_ZERO_DAYS &&
      left?.downloads > 0 &&
      right?.downloads > 0 &&
      dates[end] - dates[start - 1] === (length + 1) * DAY_MS
    ) {
      const leftBase = weekdayBaseline(downloads, dates, start - 1);
      const rightBase = weekdayBaseline(downloads, dates, end);
      for (let index = start; index < end; index++) {
        const progress =
          (dates[index] - dates[start - 1]) / (dates[end] - dates[start - 1]);
        const base = weekdayBaseline(downloads, dates, index);
        const linear =
          left.downloads + (right.downloads - left.downloads) * progress;
        const estimate =
          base && leftBase && rightBase
            ? base *
              (left.downloads / leftBase +
                (right.downloads / rightBase - left.downloads / leftBase) *
                  progress)
            : linear;
        result[index] = {
          ...result[index],
          downloads: Math.round(estimate),
          estimated: true,
        };
      }
    }
    start = end - 1;
  }

  return result;
};

export const prepareDownloadData = (
  downloads: readonly DailyDownload[],
  grouping: string,
  zeroMode: ZeroMode = "reported"
): PreparedDownload[] => {
  const reported = groupData([...downloads], grouping);
  if (zeroMode !== "estimated") {
    return reported.map((item) => ({
      ...item,
      reportedDownloads: item.downloads,
      estimatedDays: 0,
    }));
  }

  const interpolated = interpolateZeroDownloads(downloads);
  const displayed = groupData(interpolated, grouping);
  const estimatedCounts = groupData(
    interpolated.map((item) => ({
      day: item.day,
      downloads: Number(item.estimated),
    })),
    grouping
  );

  return displayed.map((item, index) => ({
    ...item,
    reportedDownloads: reported[index].downloads,
    estimatedDays: estimatedCounts[index].downloads,
  }));
};
````

## File: lib/utils.ts
````typescript
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
````

## File: providers/filters.tsx
````typescript
"use client";

import {
  parseAsArrayOf,
  parseAsString,
  parseAsStringLiteral,
  useQueryState,
} from "nuqs";
import { zeroModes } from "@/lib/download-data";

export const useTimeRange = () =>
  useQueryState("timeRange", parseAsString.withDefault("last-year"));

export const useGrouping = () =>
  useQueryState("grouping", parseAsString.withDefault("week"));

export const usePackages = () =>
  useQueryState("q", parseAsArrayOf(parseAsString, ",").withDefault([]));

export const useMetric = () =>
  useQueryState("metric", parseAsString.withDefault("downloads"));

export const useZeroMode = () =>
  useQueryState(
    "zeroMode",
    parseAsStringLiteral(zeroModes)
      .withDefault("reported")
      .withOptions({ history: "push" })
  );
````

## File: tests/fixtures/download-gaps.json
````json
{
  "sources": [
    {
      "url": "https://api.npmjs.org/downloads/range/2026-08-25:2026-09-10/react",
      "fetchedAt": "2026-09-11T16:36:51.399Z",
      "sha256": "fb241586225c893f326a864b7d7b1786158c8d5eeaeef73d4931f5a531ec6f65"
    },
    {
      "url": "https://api.npmjs.org/downloads/range/2026-08-25:2026-09-10/react-dom",
      "fetchedAt": "2026-09-11T16:36:51.431Z",
      "sha256": "93b4dd45c032227eabb84bdd5ade499e1f9df8a33d9585c90ee2daef78e2ebe1"
    },
    {
      "url": "https://api.npmjs.org/downloads/range/2026-08-25:2026-09-10/next",
      "fetchedAt": "2026-09-11T16:36:51.480Z",
      "sha256": "e9c6ea20cfba1e0a8247bb7349b7deaa07fe8cb09e911cfc63aea0c0f007dd4d"
    }
  ],
  "packages": [
    {
      "start": "2026-08-25",
      "end": "2026-09-10",
      "package": "react",
      "downloads": [
        {
          "downloads": 29424217,
          "day": "2026-08-25"
        },
        {
          "downloads": 28855243,
          "day": "2026-08-26"
        },
        {
          "downloads": 28634465,
          "day": "2026-08-27"
        },
        {
          "downloads": 25303565,
          "day": "2026-08-28"
        },
        {
          "downloads": 15006139,
          "day": "2026-08-29"
        },
        {
          "downloads": 15281994,
          "day": "2026-08-30"
        },
        {
          "downloads": 26910414,
          "day": "2026-08-31"
        },
        {
          "downloads": 30600876,
          "day": "2026-09-01"
        },
        {
          "downloads": 31679775,
          "day": "2026-09-02"
        },
        {
          "downloads": 0,
          "day": "2026-09-03"
        },
        {
          "downloads": 28047443,
          "day": "2026-09-04"
        },
        {
          "downloads": 17903937,
          "day": "2026-09-05"
        },
        {
          "downloads": 17693894,
          "day": "2026-09-06"
        },
        {
          "downloads": 0,
          "day": "2026-09-07"
        },
        {
          "downloads": 0,
          "day": "2026-09-08"
        },
        {
          "downloads": 32073030,
          "day": "2026-09-09"
        },
        {
          "downloads": 31649015,
          "day": "2026-09-10"
        }
      ]
    },
    {
      "start": "2026-08-25",
      "end": "2026-09-10",
      "package": "react-dom",
      "downloads": [
        {
          "downloads": 27638198,
          "day": "2026-08-25"
        },
        {
          "downloads": 27055461,
          "day": "2026-08-26"
        },
        {
          "downloads": 26863171,
          "day": "2026-08-27"
        },
        {
          "downloads": 23760151,
          "day": "2026-08-28"
        },
        {
          "downloads": 14202954,
          "day": "2026-08-29"
        },
        {
          "downloads": 14451736,
          "day": "2026-08-30"
        },
        {
          "downloads": 25215861,
          "day": "2026-08-31"
        },
        {
          "downloads": 28695698,
          "day": "2026-09-01"
        },
        {
          "downloads": 29718414,
          "day": "2026-09-02"
        },
        {
          "downloads": 0,
          "day": "2026-09-03"
        },
        {
          "downloads": 26272887,
          "day": "2026-09-04"
        },
        {
          "downloads": 16987576,
          "day": "2026-09-05"
        },
        {
          "downloads": 16803189,
          "day": "2026-09-06"
        },
        {
          "downloads": 0,
          "day": "2026-09-07"
        },
        {
          "downloads": 0,
          "day": "2026-09-08"
        },
        {
          "downloads": 30113801,
          "day": "2026-09-09"
        },
        {
          "downloads": 29677277,
          "day": "2026-09-10"
        }
      ]
    },
    {
      "start": "2026-08-25",
      "end": "2026-09-10",
      "package": "next",
      "downloads": [
        {
          "downloads": 9292938,
          "day": "2026-08-25"
        },
        {
          "downloads": 9099879,
          "day": "2026-08-26"
        },
        {
          "downloads": 9119588,
          "day": "2026-08-27"
        },
        {
          "downloads": 8371072,
          "day": "2026-08-28"
        },
        {
          "downloads": 5296094,
          "day": "2026-08-29"
        },
        {
          "downloads": 5373583,
          "day": "2026-08-30"
        },
        {
          "downloads": 8930505,
          "day": "2026-08-31"
        },
        {
          "downloads": 10079354,
          "day": "2026-09-01"
        },
        {
          "downloads": 0,
          "day": "2026-09-02"
        },
        {
          "downloads": 0,
          "day": "2026-09-03"
        },
        {
          "downloads": 9218297,
          "day": "2026-09-04"
        },
        {
          "downloads": 6291196,
          "day": "2026-09-05"
        },
        {
          "downloads": 6126400,
          "day": "2026-09-06"
        },
        {
          "downloads": 0,
          "day": "2026-09-07"
        },
        {
          "downloads": 0,
          "day": "2026-09-08"
        },
        {
          "downloads": 10777759,
          "day": "2026-09-09"
        },
        {
          "downloads": 10408036,
          "day": "2026-09-10"
        }
      ]
    }
  ]
}
````

## File: tests/download-data.test.mjs
````javascript
import assert from "node:assert/strict";
import { readFileSync } from "node:fs";
import { describe, test } from "node:test";
import { groupData } from "../.test-build/chart-utils.js";
import {
  interpolateZeroDownloads,
  parseZeroMode,
  prepareDownloadData,
} from "../.test-build/download-data.js";

const fixture = JSON.parse(
  readFileSync(
    new URL("./fixtures/download-gaps.json", import.meta.url),
    "utf8"
  )
);
const daily = (counts) =>
  counts.map((downloads, index) => ({
    day: `2026-09-${String(index + 1).padStart(2, "0")}`,
    downloads,
  }));
const values = (data) => data.map((item) => item.downloads);

describe("bounded zero interpolation", () => {
  test("fills one interior zero and preserves reported values", () => {
    const result = interpolateZeroDownloads(daily([1000, 0, 1200]));
    assert.deepEqual(values(result), [1000, 1100, 1200]);
    assert.equal(result[1].reportedDownloads, 0);
    assert.equal(result[1].estimated, true);
    assert.equal(result[0].estimated, false);
  });
  test("fills two consecutive days from original endpoints", () => {
    assert.deepEqual(
      values(interpolateZeroDownloads(daily([1000, 0, 0, 1600]))),
      [1000, 1200, 1400, 1600]
    );
  });
  test("fills the three-day eve outage from 2026-10-13 to 2026-10-15", () => {
    const result = interpolateZeroDownloads([
      { day: "2026-10-12", downloads: 242_780 },
      { day: "2026-10-13", downloads: 0 },
      { day: "2026-10-14", downloads: 0 },
      { day: "2026-10-15", downloads: 0 },
      { day: "2026-10-16", downloads: 306_171 },
    ]);
    assert.deepEqual(
      values(result),
      [242_780, 258_628, 274_476, 290_323, 306_171]
    );
    assert.deepEqual(
      result.map((item) => item.estimated),
      [false, true, true, true, false]
    );
  });
  test("fills up to seven consecutive days", () => {
    assert.deepEqual(
      values(interpolateZeroDownloads(daily([100, 0, 0, 0, 0, 0, 0, 0, 900]))),
      [100, 200, 300, 400, 500, 600, 700, 800, 900]
    );
  });
  test("follows the weekly pattern when nearby weeks are available", () => {
    const week = [1000, 1000, 1000, 1000, 1000, 500, 500];
    const counts = [...week, ...week, ...week, ...week];
    counts[14] = 0;
    counts[15] = 0;
    const result = interpolateZeroDownloads(daily(counts));
    assert.deepEqual(
      values(result).slice(12, 17),
      [500, 500, 1000, 1000, 1000]
    );
    assert.equal(result[14].estimated, true);
  });
  test("rounds once and handles decreasing counts", () => {
    assert.deepEqual(
      values(interpolateZeroDownloads(daily([100, 0, 0, 51]))),
      [100, 84, 67, 51]
    );
  });
  for (const input of [
    [0, 100],
    [100, 0],
    [0, 0],
    [100, 0, 0, 0, 0, 0, 0, 0, 0, 200],
    [0],
    [],
  ]) {
    test(`preserves ineligible series ${JSON.stringify(input)}`, () => {
      assert.deepEqual(values(interpolateZeroDownloads(daily(input))), input);
    });
  }
  test("never fills a missing calendar day", () => {
    const input = [
      { day: "2026-09-01", downloads: 100 },
      { day: "2026-09-03", downloads: 0 },
      { day: "2026-09-04", downloads: 200 },
    ];
    assert.deepEqual(values(interpolateZeroDownloads(input)), [100, 0, 200]);
  });
  test("does not mutate frozen input or positive values", () => {
    const input = Object.freeze(
      daily([100, 0, 200, 0, 400]).map(Object.freeze)
    );
    assert.deepEqual(
      values(interpolateZeroDownloads(input)),
      [100, 150, 200, 300, 400]
    );
    assert.deepEqual(values(input), [100, 0, 200, 0, 400]);
  });
  for (const count of [Number.NaN, Number.POSITIVE_INFINITY, -1, 1.5]) {
    test(`rejects invalid count ${count}`, () => {
      assert.throws(
        () => interpolateZeroDownloads(daily([100, count, 200])),
        RangeError
      );
    });
  }
  for (const day of ["invalid", "2026-02-30", "2026-9-01"]) {
    test(`rejects invalid UTC date ${day}`, () => {
      assert.throws(
        () => interpolateZeroDownloads([{ day, downloads: 1 }]),
        RangeError
      );
    });
  }
  test("rejects duplicate and out-of-order dates", () => {
    const input = daily([100, 0, 200]);
    assert.throws(
      () => interpolateZeroDownloads([...input].reverse()),
      RangeError
    );
    assert.throws(
      () => interpolateZeroDownloads([input[0], input[0]]),
      RangeError
    );
  });
});

describe("data policy and grouping", () => {
  for (const mode of [
    null,
    undefined,
    "garbage",
    "reported",
    "ESTIMATED",
    true,
  ]) {
    test(`defaults invalid mode ${mode} to reported`, () =>
      assert.equal(parseZeroMode(mode), "reported"));
  }
  test("recognizes explicit opt-in", () =>
    assert.equal(parseZeroMode("estimated"), "estimated"));
  for (const grouping of ["day", "week", "month"]) {
    test(`reported ${grouping} matches existing grouping`, () => {
      const input = daily([1000, 0, 0, 1600]);
      assert.deepEqual(
        prepareDownloadData(input, grouping).map(({ date, downloads }) => ({
          date,
          downloads,
        })),
        groupData(input, grouping)
      );
    });
  }
  for (const grouping of ["week", "month"]) {
    test(`estimates before ${grouping} aggregation and preserves provenance`, () => {
      const result = prepareDownloadData(
        daily([1000, 0, 0, 1600]),
        grouping,
        "estimated"
      )[0];
      assert.equal(result.downloads, 5200);
      assert.equal(result.reportedDownloads, 2600);
      assert.equal(result.estimatedDays, 2);
    });
  }
  test("retains legitimate low-volume zeros by default", () => {
    assert.deepEqual(
      values(prepareDownloadData(daily([1, 0, 1]), "day")),
      [1, 0, 1]
    );
  });
  test("captured next gap is independent of positive react data", () => {
    const next = fixture.packages.find((pkg) => pkg.package === "next");
    const react = fixture.packages.find((pkg) => pkg.package === "react");
    const nextPoint = prepareDownloadData(
      next.downloads,
      "day",
      "estimated"
    ).find((p) => p.date === "2026-09-02");
    const reactPoint = prepareDownloadData(
      react.downloads,
      "day",
      "estimated"
    ).find((p) => p.date === "2026-09-02");
    assert.deepEqual(nextPoint, {
      date: "2026-09-02",
      downloads: 10_834_833,
      reportedDownloads: 0,
      estimatedDays: 1,
    });
    assert.deepEqual(reactPoint, {
      date: "2026-09-02",
      downloads: 31_679_775,
      reportedDownloads: 31_679_775,
      estimatedDays: 0,
    });
  });
});
````

## File: tests/serve-fixture.mjs
````javascript
import { readFileSync } from "node:fs";
import { createServer } from "node:http";
import next from "next";

if (process.env.NODE_ENV === "production") {
  throw new Error("Fixture server is development-only");
}

const fixture = JSON.parse(
  readFileSync(
    new URL("./fixtures/download-gaps.json", import.meta.url),
    "utf8"
  )
);
const originalFetch = globalThis.fetch;
globalThis.fetch = (input, init) => {
  const url = new URL(input instanceof Request ? input.url : String(input));
  if (url.hostname === "api.npmjs.org") {
    const pkg = fixture.packages.find(
      (item) =>
        url.pathname ===
        `/downloads/range/${item.start}:${item.end}/${item.package}`
    );
    if (!pkg) {
      throw new Error(`No frozen response for ${url.pathname}`);
    }
    console.info(`Frozen npm response: ${pkg.package}`);
    return Promise.resolve(Response.json(pkg));
  }
  return originalFetch(input, init);
};

const port = Number(process.env.PORT || 3001);
const app = next({
  dev: true,
  webpack: true,
  dir: process.cwd(),
  hostname: "127.0.0.1",
  port,
});
await app.prepare();
const handle = app.getRequestHandler();
createServer((request, response) => handle(request, response)).listen(
  port,
  "127.0.0.1",
  () => {
    console.info(`Frozen-data preview: http://localhost:${port}`);
  }
);
````

## File: tests/verify-preview.mjs
````javascript
import assert from "node:assert/strict";
import { test } from "node:test";

const base = process.env.PREVIEW_URL || "http://localhost:3001";
const circlePattern = /<circle /g;
const dataMarkerPattern = /<circle [^>]*><title>/g;
const params = new URLSearchParams({
  q: "react,react-dom,next",
  timeRange: "2026-08-25:2026-09-10",
  grouping: "day",
});

test("frozen preview preserves zero-mode semantics across exports", async () => {
  const get = async (path, extra = {}) => {
    const query = new URLSearchParams(params);
    for (const [key, value] of Object.entries(extra)) {
      query.set(key, value);
    }
    const response = await fetch(`${base}${path}?${query}`);
    assert.equal(response.status, 200, `${path}: ${response.status}`);
    return response;
  };
  const reported = await (await get("/svg")).text();
  const estimated = await (await get("/svg", { zeroMode: "estimated" })).text();
  assert.equal((reported.match(circlePattern) || []).length, 0);
  assert.equal((estimated.match(dataMarkerPattern) || []).length, 10);
  assert.ok(estimated.includes("9792335 downloads; 0 reported downloads"));
  assert.ok(estimated.includes("Includes estimated downloads"));
  assert.ok(!reported.includes("Includes estimated downloads"));
  assert.ok(!(estimated.includes("NaN") || estimated.includes("Infinity")));
  assert.equal(
    await (await get("/svg", { zeroMode: "invalid" })).text(),
    reported
  );

  const share = await (
    await get("/svg", { zeroMode: "estimated", metric: "share" })
  ).text();
  assert.equal((share.match(dataMarkerPattern) || []).length, 12);
  assert.ok(share.includes("share denominators include estimates"));
  assert.ok(share.includes("100%"));

  for (const path of ["/", "/react,react-dom,next"]) {
    const html = await (await get(path, { zeroMode: "estimated" })).text();
    assert.ok(html.includes("zeroMode=estimated"));
    assert.ok(html.includes("og:image"));
  }
  for (const metric of ["downloads", "share"]) {
    const response = await get("/og", { zeroMode: "estimated", metric });
    assert.ok(response.headers.get("content-type").includes("image/png"));
    const png = Buffer.from(await response.arrayBuffer());
    assert.equal(png.subarray(1, 4).toString(), "PNG");
    assert.equal(png.readUInt32BE(16), 1200);
    assert.equal(png.readUInt32BE(20), 628);
  }
});
````

## File: .gitattributes
````
# Auto detect text files and perform LF normalization
* text=auto
````

## File: .gitignore
````
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.*
.yarn/*
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/versions

# testing
/coverage
/.test-build/

# next.js
/.next/
/out/

# production
/build

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# env files (can opt-in for committing if needed)
.env*

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts
````

## File: biome.jsonc
````json
{
  "$schema": "./node_modules/@biomejs/biome/configuration_schema.json",
  "extends": ["ultracite/core", "ultracite/react", "ultracite/next"],
  "files": {
    "includes": [
      "**/*",
      "!components/ui",
      "!lib/utils.ts",
      "!hooks/use-mobile.ts"
    ]
  }
}
````

## File: components.json
````json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "iconLibrary": "lucide",
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "registries": {}
}
````

## File: next.config.ts
````typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    formats: ["image/avif", "image/webp"],
  },

  experimental: {
    turbopackFileSystemCacheForDev: true,
  },
};

export default nextConfig;
````

## File: package.json
````json
{
  "name": "npm.bet",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "test": "tsc -p tsconfig.test.json && node --test tests/*.test.mjs",
    "check": "ultracite check",
    "fix": "ultracite fix"
  },
  "dependencies": {
    "@hookform/resolvers": "^5.2.2",
    "@upstash/redis": "^1.36.2",
    "@vercel/analytics": "^1.6.1",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "cmdk": "^1.1.1",
    "date-fns": "^4.1.0",
    "embla-carousel-react": "^8.6.0",
    "html-to-image": "^1.11.13",
    "input-otp": "^1.4.2",
    "lucide-react": "^0.563.0",
    "next": "16.2.3",
    "next-themes": "^0.4.6",
    "nuqs": "^2.8.8",
    "radix-ui": "^1.4.3",
    "react": "19.2.4",
    "react-day-picker": "^9.12.0",
    "react-dom": "19.2.4",
    "react-hook-form": "^7.71.1",
    "react-resizable-panels": "^3.0.6",
    "recharts": "2.15.4",
    "sonner": "^2.0.7",
    "swr": "^2.4.0",
    "tailwind-merge": "^3.4.0",
    "vaul": "^1.1.2",
    "zod": "^4.3.6"
  },
  "devDependencies": {
    "@biomejs/biome": "2.3.14",
    "@tailwindcss/postcss": "^4",
    "@types/node": "^25",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "tailwindcss": "^4",
    "tw-animate-css": "^1.4.0",
    "typescript": "^5",
    "ultracite": "7.1.4"
  }
}
````

## File: postcss.config.mjs
````javascript
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
````

## File: README.md
````markdown
# npm.bet

A beautiful, interactive npm package download statistics visualizer built with Next.js 16, React 19, and Recharts.

## Features

- **Multi-package comparison** - Compare download trends across multiple npm packages simultaneously
- **Flexible time ranges** - View data from the last week to all-time statistics
- **Smart grouping** - Aggregate data by day, week, or month for better insights
- **Interactive charts** - Built with Recharts for smooth, responsive visualizations
- **Screenshot export** - Download charts as high-resolution PNG images
- **SVG embed** - Generate embeddable SVG charts with code snippets (Markdown, HTML, or direct URL) for your documentation
- **Dark mode support** - Seamless theme switching with next-themes
- **Real-time search** - Search and add npm packages with autocomplete
- **Responsive design** - Works perfectly on desktop and mobile devices

## Tech Stack

- **Framework**: [Next.js 16](https://nextjs.org) with App Router
- **UI Library**: [React 19](https://react.dev)
- **Styling**: [Tailwind CSS 4](https://tailwindcss.com)
- **Charts**: [Recharts](https://recharts.org)
- **Date Handling**: [date-fns](https://date-fns.org) with timezone support
- **Data Fetching**: [SWR](https://swr.vercel.app)
- **State Management**: [nuqs](https://nuqs.47ng.com) for URL state
- **Code Quality**: [Ultracite](https://github.com/biomejs/ultracite) (Biome preset)
- **UI Components**: Custom components built on [Radix UI](https://www.radix-ui.com)

## Getting Started

### Prerequisites

- Node.js 20 or higher
- pnpm

### Installation

```bash
# Clone the repository
git clone https://github.com/vercel/npm.bet.git
cd npm.bet

# Install dependencies
pnpm install

# Run the development server
pnpm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the application.

### Available Scripts

- `pnpm dev` - Start the development server
- `pnpm build` - Build the production application
- `pnpm test` - Compile the data utilities and run deterministic Node.js tests
- `pnpm check` - Check for code quality issues
- `pnpm fix` - Automatically fix code quality issues

## Usage

1. **Search for packages**: Use the search bar to find npm packages by name
2. **Select time range**: Choose from predefined ranges (last week, month, year, 2 years, 5 years, or all-time)
3. **Adjust grouping**: Switch between daily, weekly, or monthly aggregation
4. **Compare packages**: Add multiple packages to compare their download trends
5. **Export chart**: Click the camera icon to download the chart as a PNG
6. **Embed SVG**: Click the code icon to get embeddable SVG snippets for your README or website

### Zero-download gaps

Charts show npm-reported counts by default. Enable **Interpolate zeros** to estimate internal gaps of up to seven consecutive zero-download days between positive reported values. Each estimate uses the same weekday in up to three weeks before and after, scaled to the gap's neighbors, and falls back to a straight line when that history is missing. The setting is preserved as `zeroMode=estimated` in shared URLs, SVG embeds, and OG previews. Missing dates, leading/trailing zeros, and longer gaps are not filled. Weeks start on Sunday and grouping uses UTC, matching npm's daily dates.

Estimates use linear interpolation between the reported endpoints, rounded to whole downloads. Hollow markers identify affected points; tooltips retain the reported counts. Weekly/monthly totals and share percentages use the selected mode and disclose estimated inputs. A zero is not proof of an outage, so interpolation remains opt-in.

Run `pnpm test` for the pure transformation tests. `tests/fixtures/download-gaps.json` contains frozen real responses and source timestamps for reproducible comparisons. A development-only fixture server is available with `pnpm exec node tests/serve-fixture.mjs`; run it in an isolated copy when another dev server is active. It listens on port 3001 by default and supports the fixture's exact package/date range. `pnpm exec node tests/verify-preview.mjs` verifies that preview's SVG, share, metadata, and OG responses.

## Project Structure

```
npm.bet/
├── actions/          # Server actions for data fetching
│   └── package/      # Package search and download data
├── app/              # Next.js App Router pages
├── components/       # React components
│   ├── ui/           # Reusable UI components (Radix-based)
│   ├── chart.tsx     # Main chart component
│   ├── header.tsx    # Application header
│   └── ...
├── hooks/            # Custom React hooks
├── lib/              # Utility functions
└── providers/        # React context providers
```

## API Integration

This project uses the official [npm Registry API](https://github.com/npm/registry/blob/master/docs/download-counts.md) to fetch package download statistics:

- Package search: `https://registry.npmjs.com/-/v1/search`
- Download stats: `https://api.npmjs.org/downloads/range/{period}/{package}`

Data is cached for 1 hour to reduce API load and improve performance.

## Code Quality

This project uses **Ultracite**, a zero-config Biome preset for automated code formatting and linting. Key standards include:

- TypeScript strict mode with explicit types
- React 19 best practices (ref as prop, no forwardRef)
- Accessibility-first approach (ARIA attributes, semantic HTML)
- Modern JavaScript/TypeScript patterns
- No console.log or debugger statements in production

Run `npx ultracite doctor` to verify your setup.

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Run `npx ultracite fix` to ensure code quality
5. Commit your changes with a descriptive message
6. Push to your branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## License

This project is open source and available under the MIT License.

## Acknowledgments

- npm Registry API for providing download statistics
- Vercel for hosting and deployment
- The React and Next.js teams for amazing frameworks
- Radix UI for accessible component primitives

---

Built with ❤️ by [Vercel](https://github.com/vercel)
````

## File: tsconfig.json
````json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    },
    "strictNullChecks": true
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts",
    "**/*.mts"
  ],
  "exclude": ["node_modules"]
}
````

## File: tsconfig.test.json
````json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "lib",
    "outDir": ".test-build",
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["lib/chart-utils.ts", "lib/download-data.ts"]
}
````
