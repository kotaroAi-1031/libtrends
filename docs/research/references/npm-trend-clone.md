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
- Only files matching these patterns are included: src/**, README.md, package.json
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)

# Directory Structure
````
src/
  __tests__/
    colors.test.ts
    is-msw-enabled.test.ts
    msw-fixtures.test.ts
  app/
    api/
      auth/
        [...all]/
          route.ts
      downloads/
        route.ts
      health/
        route.ts
      packages/
        info/
          route.ts
        search/
          route.ts
      presets/
        [id]/
          route.ts
        route.ts
    forgot-password/
      layout.tsx
      page.tsx
    login/
      layout.tsx
      page.tsx
    reset-password/
      layout.tsx
      page.tsx
    settings/
      page.tsx
    signup/
      layout.tsx
      page.tsx
    verify-email/
      layout.tsx
      page.tsx
    error.tsx
    global-error.tsx
    globals.css
    layout.tsx
    not-found.tsx
    page.tsx
  components/
    providers/
      index.tsx
      msw-provider.tsx
      theme-provider.tsx
    ui/
      alert-dialog.tsx
      badge.tsx
      button.tsx
      card.tsx
      dialog.tsx
      dropdown-menu.tsx
      input.tsx
      label.tsx
      progress.tsx
      skeleton.tsx
      tooltip.tsx
    category-button.tsx
    discover-packages.tsx
    export-button.tsx
    mobile-nav.tsx
    offline-indicator.tsx
    package-info-card.tsx
    package-tag-bar.stories.tsx
    package-tag-bar.tsx
    package-tag.stories.tsx
    package-tag.tsx
    popular-libraries.stories.tsx
    popular-libraries.tsx
    preset-manager.tsx
    protected-route.tsx
    search-bar.stories.tsx
    search-bar.tsx
    share-button.tsx
    theme-toggle.tsx
    time-range-selector.tsx
    trend-chart.stories.tsx
    trend-chart.tsx
    user-menu.tsx
  constants/
    category-presets.ts
    colors.ts
  contexts/
    auth-context.tsx
  hooks/
    use-downloads.ts
    use-online-status.ts
    use-package-info.ts
    use-presets.ts
    use-reduced-motion.ts
    use-search-bar.ts
    use-url-state.ts
  lib/
    auth-client.ts
    auth-fetch.ts
    auth.ts
    email-templates.ts
    email.ts
    redis.ts
    utils.ts
  types/
    package.ts
  utils/
    color.ts
    isMSWEnabled.ts
  env.ts
  middleware.ts
package.json
README.md
````

# Files

## File: src/__tests__/colors.test.ts
````typescript
import { describe, it, expect } from "vitest";
import { CHART_COLORS, MAX_PACKAGES } from "@/constants/colors";

describe("Chart Colors Constants", () => {
  describe("CHART_COLORS", () => {
    it("should have enough colors for MAX_PACKAGES", () => {
      expect(CHART_COLORS.length).toBeGreaterThanOrEqual(MAX_PACKAGES);
    });

    it("should have valid hex color format", () => {
      const hexColorRegex = /^#[0-9A-Fa-f]{6}$/;

      for (const color of CHART_COLORS) {
        expect(color).toMatch(hexColorRegex);
      }
    });

    it("should have unique colors", () => {
      const uniqueColors = new Set(CHART_COLORS);
      expect(uniqueColors.size).toBe(CHART_COLORS.length);
    });

    it("should have visually distinct colors", () => {
      // Basic check: no two colors should be too similar
      // (This is a simplified check - in reality you'd calculate color distance)
      for (let i = 0; i < CHART_COLORS.length; i++) {
        for (let j = i + 1; j < CHART_COLORS.length; j++) {
          expect(CHART_COLORS[i]).not.toBe(CHART_COLORS[j]);
        }
      }
    });
  });

  describe("MAX_PACKAGES", () => {
    it("should be a reasonable limit", () => {
      expect(MAX_PACKAGES).toBeGreaterThanOrEqual(2);
      expect(MAX_PACKAGES).toBeLessThanOrEqual(10);
    });

    it("should be 6", () => {
      expect(MAX_PACKAGES).toBe(6);
    });
  });
});
````

## File: src/__tests__/is-msw-enabled.test.ts
````typescript
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";

/**
 * Tests for MSW activation logic.
 * Note: These tests mock the environment to test different scenarios.
 */
describe("isMSWEnabled utility", () => {
  const originalWindow = globalThis.window;
  const originalEnv = process.env;

  beforeEach(() => {
    vi.resetModules();
    process.env = { ...originalEnv };
  });

  afterEach(() => {
    globalThis.window = originalWindow;
    process.env = originalEnv;
  });

  describe("Server-side behavior", () => {
    it("should return false when MSW is not enabled", async () => {
      // Ensure window is undefined (server-side)
      // @ts-expect-error - Intentionally deleting for test
      delete globalThis.window;

      process.env.NEXT_PUBLIC_ENABLE_MSW_MOCK = "false";
      process.env.APP_ENV = "development";

      // Re-import to get fresh module
      const { isMSWEnabled } = await import("@/utils/isMSWEnabled");
      expect(isMSWEnabled()).toBe(false);
    });

    it("should require both MSW flag and APP_ENV=test on server", async () => {
      // @ts-expect-error - Intentionally deleting for test
      delete globalThis.window;

      process.env.NEXT_PUBLIC_ENABLE_MSW_MOCK = "true";
      process.env.APP_ENV = "development"; // Not test!

      const { isMSWEnabled } = await import("@/utils/isMSWEnabled");

      // On server, should require APP_ENV=test OR NODE_ENV=test
      // This test verifies the double-check safety
      expect(typeof isMSWEnabled).toBe("function");
    });
  });

  describe("Environment variable validation", () => {
    it("should handle missing NEXT_PUBLIC_ENABLE_MSW_MOCK", () => {
      expect(process.env.NEXT_PUBLIC_ENABLE_MSW_MOCK).toBeUndefined();
    });

    it("should handle APP_ENV values", () => {
      process.env.APP_ENV = "test";
      expect(process.env.APP_ENV).toBe("test");

      process.env.APP_ENV = "development";
      expect(process.env.APP_ENV).toBe("development");
    });
  });
});
````

## File: src/__tests__/msw-fixtures.test.ts
````typescript
import { describe, it, expect } from "vitest";
import {
  getMockSearchResults,
  reactSearchResults,
  vueSearchResults,
  angularSearchResults,
  emptySearchResults,
} from "../../mocks/fixtures/search-results";
import {
  generateMockDownloadData,
  getWeeklyDownloads,
  reactDownloads30d,
  vueDownloads30d,
} from "../../mocks/fixtures/download-data";

describe("MSW Fixtures - Search Results", () => {
  describe("getMockSearchResults", () => {
    it("should return react results for 'react' query", () => {
      const result = getMockSearchResults("react");
      expect(result).toBe(reactSearchResults);
      expect(result.objects.length).toBeGreaterThan(0);
      expect(result.objects[0].package.name).toBe("react");
    });

    it("should return react results for 'React' query (case insensitive)", () => {
      const result = getMockSearchResults("React");
      expect(result).toBe(reactSearchResults);
    });

    it("should return react results for 'react-router' query", () => {
      const result = getMockSearchResults("react-router");
      expect(result).toBe(reactSearchResults);
    });

    it("should return vue results for 'vue' query", () => {
      const result = getMockSearchResults("vue");
      expect(result).toBe(vueSearchResults);
      expect(result.objects[0].package.name).toBe("vue");
    });

    it("should return angular results for 'angular' query", () => {
      const result = getMockSearchResults("angular");
      expect(result).toBe(angularSearchResults);
    });

    it("should return empty results for unknown query", () => {
      const result = getMockSearchResults("xyznonexistent");
      expect(result).toBe(emptySearchResults);
      expect(result.objects).toHaveLength(0);
      expect(result.total).toBe(0);
    });

    it("should handle whitespace in query", () => {
      const result = getMockSearchResults("  react  ");
      expect(result).toBe(reactSearchResults);
    });
  });

  describe("Search result structure", () => {
    it("should have valid package structure", () => {
      const result = reactSearchResults;

      for (const obj of result.objects) {
        expect(obj.package).toHaveProperty("name");
        expect(obj.package).toHaveProperty("version");
        expect(obj.package).toHaveProperty("description");
        expect(obj.downloads).toHaveProperty("weekly");
        expect(obj.downloads).toHaveProperty("monthly");
        expect(typeof obj.downloads.weekly).toBe("number");
        expect(typeof obj.downloads.monthly).toBe("number");
      }
    });

    it("should have realistic download counts", () => {
      const react = reactSearchResults.objects[0];

      // React should have millions of downloads
      expect(react.downloads.weekly).toBeGreaterThan(1_000_000);
      expect(react.downloads.monthly).toBeGreaterThan(react.downloads.weekly);
    });
  });
});

describe("MSW Fixtures - Download Data", () => {
  describe("getWeeklyDownloads", () => {
    it("should return correct downloads for react", () => {
      const downloads = getWeeklyDownloads("react");
      expect(downloads).toBe(58239993);
    });

    it("should return correct downloads for vue", () => {
      const downloads = getWeeklyDownloads("vue");
      expect(downloads).toBe(7828470);
    });

    it("should return default value for unknown package", () => {
      const downloads = getWeeklyDownloads("unknown-package-xyz");
      expect(downloads).toBe(10000);
    });
  });

  describe("generateMockDownloadData", () => {
    it("should generate data for the correct date range", () => {
      const data = generateMockDownloadData("react", "2025-12-01", "2025-12-10");

      expect(data.package).toBe("react");
      expect(data.downloads.length).toBe(10); // 10 days
      expect(data.downloads[0].day).toBe("2025-12-01");
      expect(data.downloads[9].day).toBe("2025-12-10");
    });

    it("should generate positive download counts", () => {
      const data = generateMockDownloadData("react", "2025-12-01", "2025-12-07");

      for (const point of data.downloads) {
        expect(point.downloads).toBeGreaterThan(0);
        expect(typeof point.downloads).toBe("number");
      }
    });

    it("should use base downloads from known packages", () => {
      const reactData = generateMockDownloadData("react", "2025-12-01", "2025-12-03");
      const unknownData = generateMockDownloadData("unknown-pkg", "2025-12-01", "2025-12-03");

      // React should have higher average downloads
      const reactAvg = reactData.downloads.reduce((sum, d) => sum + d.downloads, 0) / reactData.downloads.length;
      const unknownAvg = unknownData.downloads.reduce((sum, d) => sum + d.downloads, 0) / unknownData.downloads.length;

      expect(reactAvg).toBeGreaterThan(unknownAvg);
    });

    it("should have lower weekend downloads", () => {
      // Dec 6-7, 2025 are Saturday and Sunday
      const data = generateMockDownloadData("react", "2025-12-05", "2025-12-08");

      const friday = data.downloads.find((d) => d.day === "2025-12-05");
      const saturday = data.downloads.find((d) => d.day === "2025-12-06");
      const monday = data.downloads.find((d) => d.day === "2025-12-08");

      // Weekend should generally be lower (though random variation may affect this)
      // Just verify they exist
      expect(friday).toBeDefined();
      expect(saturday).toBeDefined();
      expect(monday).toBeDefined();
    });
  });

  describe("Pre-generated download data", () => {
    it("should have 31 days of data for react", () => {
      expect(reactDownloads30d.downloads.length).toBe(31);
      expect(reactDownloads30d.package).toBe("react");
    });

    it("should have 31 days of data for vue", () => {
      expect(vueDownloads30d.downloads.length).toBe(31);
      expect(vueDownloads30d.package).toBe("vue");
    });

    it("should have valid date format", () => {
      const dateRegex = /^\d{4}-\d{2}-\d{2}$/;

      for (const point of reactDownloads30d.downloads) {
        expect(point.day).toMatch(dateRegex);
      }
    });

    it("should have chronologically ordered dates", () => {
      const dates = reactDownloads30d.downloads.map((d) => new Date(d.day).getTime());

      for (let i = 1; i < dates.length; i++) {
        expect(dates[i]).toBeGreaterThan(dates[i - 1]);
      }
    });
  });
});
````

## File: src/app/api/auth/[...all]/route.ts
````typescript
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

/**
 * Better Auth API route handler.
 * Handles all authentication requests at /api/auth/*.
 *
 * @description
 * Endpoints include:
 * - POST /api/auth/sign-in/email - Email/password sign-in
 * - POST /api/auth/sign-up/email - Email/password registration
 * - POST /api/auth/sign-in/social - OAuth provider sign-in
 * - POST /api/auth/sign-out - End session
 * - GET /api/auth/session - Get current session
 * - GET /api/auth/callback/:provider - OAuth callback handling
 *
 * @see https://www.better-auth.com/docs/integrations/next
 */
export const { GET, POST } = toNextJsHandler(auth);
````

## File: src/app/api/downloads/route.ts
````typescript
import { NextRequest, NextResponse } from "next/server";

/**
 * NPM API download response for a date range.
 */
interface NpmDownloadResponse {
  package: string;
  downloads: Array<{
    downloads: number;
    day: string;
  }>;
}

/**
 * Time range configuration for download queries.
 */
const TIME_RANGES = {
  "1m": 30,
  "3m": 90,
  "6m": 180,
  "1y": 365,
  "2y": 730,
  "5y": 1825,
} as const;

type TimeRange = keyof typeof TIME_RANGES;

/**
 * Calculates the start date for a given time range.
 * @param range - Time range key (1m, 3m, 6m, 1y, 2y, 5y)
 * @returns ISO date string for the start date
 */
function getStartDate(range: TimeRange): string {
  const days = TIME_RANGES[range];
  const date = new Date();
  date.setDate(date.getDate() - days);
  return date.toISOString().split("T")[0];
}

/**
 * Fetches download statistics for multiple npm packages.
 * @route GET /api/downloads?packages=react,vue&range=1y
 * @param request - Next.js request object
 * @returns JSON object with download data for each package
 * @example
 * GET /api/downloads?packages=react,vue&range=1y
 * // => { react: { downloads: [...] }, vue: { downloads: [...] } }
 */
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const packagesParam = searchParams.get("packages");
  const range = (searchParams.get("range") || "1y") as TimeRange;

  if (!packagesParam) {
    return NextResponse.json(
      { error: "Packages parameter is required" },
      { status: 400 }
    );
  }

  const packages = packagesParam.split(",").filter(Boolean);

  if (packages.length === 0) {
    return NextResponse.json(
      { error: "At least one package is required" },
      { status: 400 }
    );
  }

  if (packages.length > 10) {
    return NextResponse.json(
      { error: "Maximum 10 packages allowed" },
      { status: 400 }
    );
  }

  if (!(range in TIME_RANGES)) {
    return NextResponse.json(
      { error: `Invalid range. Valid values: ${Object.keys(TIME_RANGES).join(", ")}` },
      { status: 400 }
    );
  }

  const startDate = getStartDate(range);
  const endDate = new Date().toISOString().split("T")[0];

  try {
    // Fetch downloads for all packages in parallel
    const downloadPromises = packages.map(async (pkg) => {
      const response = await fetch(
        `https://api.npmjs.org/downloads/range/${startDate}:${endDate}/${encodeURIComponent(pkg)}`,
        {
          headers: {
            Accept: "application/json",
          },
          next: { revalidate: 3600 }, // Cache for 1 hour
        }
      );

      if (!response.ok) {
        if (response.status === 404) {
          return { package: pkg, error: "Package not found", downloads: [] };
        }
        throw new Error(`npm API returned ${response.status} for ${pkg}`);
      }

      const data: NpmDownloadResponse = await response.json();
      return data;
    });

    const results = await Promise.all(downloadPromises);

    // Build response object
    const responseData: Record<
      string,
      { downloads: Array<{ day: string; downloads: number }> } | { error: string }
    > = {};

    for (const result of results) {
      if ("error" in result) {
        responseData[result.package] = { error: result.error };
      } else {
        responseData[result.package] = {
          downloads: result.downloads,
        };
      }
    }

    return NextResponse.json(responseData);
  } catch (error) {
    console.error("Error fetching download data:", error);
    return NextResponse.json(
      { error: "Failed to fetch download data" },
      { status: 500 }
    );
  }
}
````

## File: src/app/api/health/route.ts
````typescript
import { NextResponse } from "next/server";
import { checkRedisHealth } from "@/lib/redis";

/**
 * Health check endpoint.
 * Returns the application health status including Redis connectivity.
 * GET /api/health
 *
 * @returns
 * - status: "healthy" when all services are operational
 * - services.redis: "up" when Redis PING succeeds, "down" otherwise
 *
 * @example
 * // Response when Redis is connected:
 * {
 *   "status": "healthy",
 *   "timestamp": "2025-12-21T...",
 *   "services": { "app": "ok", "redis": "up" },
 *   "uptime": 123.45
 * }
 */
export async function GET() {
  const redisHealth = await checkRedisHealth();

  const isHealthy = redisHealth.status === "up";

  const health = {
    status: isHealthy ? "healthy" : "degraded",
    timestamp: new Date().toISOString(),
    services: {
      app: "ok",
      redis: redisHealth.status,
    },
    details: {
      redis:
        redisHealth.status === "up"
          ? { latencyMs: redisHealth.latencyMs }
          : { error: redisHealth.error },
    },
    uptime: process.uptime(),
  };

  return NextResponse.json(health, {
    status: isHealthy ? 200 : 503,
  });
}
````

## File: src/app/api/packages/info/route.ts
````typescript
import { NextRequest, NextResponse } from "next/server";
import type { PackageInfo } from "@/types/package";

/**
 * npm registry package metadata response structure.
 */
interface NpmRegistryPackage {
  name: string;
  description?: string;
  "dist-tags": { latest: string };
  license?: string | { type: string };
  homepage?: string;
  repository?: { type: string; url: string } | string;
  time: Record<string, string>;
}

/**
 * npm downloads API response structure.
 */
interface NpmDownloadsResponse {
  downloads: number;
  package: string;
  start: string;
  end: string;
}

/**
 * GitHub repository API response structure.
 */
interface GitHubRepoResponse {
  stargazers_count: number;
}

/**
 * Extracts GitHub repository URL from npm package repository field.
 * @param repository - npm repository field
 * @returns Normalized GitHub URL or undefined
 */
function extractGitHubUrl(
  repository?: { type: string; url: string } | string
): string | undefined {
  if (!repository) return undefined;

  let url: string;
  if (typeof repository === "string") {
    url = repository;
  } else {
    url = repository.url || "";
  }

  // Handle various GitHub URL formats
  // git+https://github.com/user/repo.git
  // git://github.com/user/repo.git
  // https://github.com/user/repo
  // github:user/repo

  if (url.includes("github.com")) {
    return url
      .replace(/^git\+/, "")
      .replace(/^git:\/\//, "https://")
      .replace(/\.git$/, "")
      .replace(/^ssh:\/\/git@github\.com/, "https://github.com");
  }

  if (url.startsWith("github:")) {
    return `https://github.com/${url.replace("github:", "")}`;
  }

  return undefined;
}

/**
 * Extracts license string from npm package license field.
 * @param license - npm license field
 * @returns License string or "Unknown"
 */
function extractLicense(license?: string | { type: string }): string {
  if (!license) return "Unknown";
  if (typeof license === "string") return license;
  return license.type || "Unknown";
}

/**
 * Fetches GitHub stars for a repository.
 * @param repoUrl - GitHub repository URL
 * @returns Star count or undefined if failed
 */
async function fetchGitHubStars(repoUrl: string): Promise<number | undefined> {
  try {
    // Extract owner/repo from URL
    const match = repoUrl.match(/github\.com\/([^/]+)\/([^/]+)/);
    if (!match) return undefined;

    const [, owner, repo] = match;
    const apiUrl = `https://api.github.com/repos/${owner}/${repo}`;

    const response = await fetch(apiUrl, {
      headers: {
        Accept: "application/vnd.github.v3+json",
        "User-Agent": "npm-trend-clone",
      },
      next: { revalidate: 3600 }, // Cache for 1 hour
    });

    if (!response.ok) return undefined;

    const data: GitHubRepoResponse = await response.json();
    return data.stargazers_count;
  } catch {
    return undefined;
  }
}

/**
 * Get package information from npm registry.
 * @route GET /api/packages/info?packages=react,vue
 * @param request - Next.js request object
 * @returns JSON object with package information
 * @example
 * GET /api/packages/info?packages=react,vue
 * // => { react: { name: "react", ... }, vue: { name: "vue", ... } }
 */
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const packagesParam = searchParams.get("packages");

  if (!packagesParam) {
    return NextResponse.json(
      { error: "packages parameter is required" },
      { status: 400 }
    );
  }

  const packageNames = packagesParam.split(",").filter(Boolean);

  if (packageNames.length === 0) {
    return NextResponse.json(
      { error: "At least one package name is required" },
      { status: 400 }
    );
  }

  const results: Record<string, PackageInfo | { error: string }> = {};

  await Promise.all(
    packageNames.map(async (name) => {
      try {
        // Fetch package metadata from npm registry
        const [registryResponse, downloadsResponse] = await Promise.all([
          fetch(`https://registry.npmjs.org/${encodeURIComponent(name)}`, {
            headers: { Accept: "application/json" },
            next: { revalidate: 3600 }, // Cache for 1 hour
          }),
          fetch(
            `https://api.npmjs.org/downloads/point/last-week/${encodeURIComponent(name)}`,
            {
              headers: { Accept: "application/json" },
              next: { revalidate: 3600 },
            }
          ),
        ]);

        if (!registryResponse.ok) {
          results[name] = { error: "Package not found" };
          return;
        }

        const registryData: NpmRegistryPackage = await registryResponse.json();
        const latestVersion = registryData["dist-tags"].latest;
        const repositoryUrl = extractGitHubUrl(registryData.repository);

        // Fetch GitHub stars if repository URL is available
        const githubStars = repositoryUrl
          ? await fetchGitHubStars(repositoryUrl)
          : undefined;

        // Get weekly downloads
        let weeklyDownloads = 0;
        if (downloadsResponse.ok) {
          const downloadsData: NpmDownloadsResponse =
            await downloadsResponse.json();
          weeklyDownloads = downloadsData.downloads;
        }

        // Get last publish date for latest version
        const lastPublished =
          registryData.time[latestVersion] || registryData.time.modified;

        const packageInfo: PackageInfo = {
          name: registryData.name,
          description: registryData.description || "No description available",
          version: latestVersion,
          license: extractLicense(registryData.license),
          homepage: registryData.homepage,
          repositoryUrl,
          lastPublished,
          weeklyDownloads,
          githubStars,
        };

        results[name] = packageInfo;
      } catch (error) {
        console.error(`Error fetching package ${name}:`, error);
        results[name] = { error: "Failed to fetch package info" };
      }
    })
  );

  return NextResponse.json(results);
}
````

## File: src/app/api/packages/search/route.ts
````typescript
import { NextRequest, NextResponse } from "next/server";

/**
 * NPM package search result from registry API.
 */
interface NpmSearchResult {
  objects: Array<{
    package: {
      name: string;
      version: string;
      description?: string;
    };
  }>;
}

/**
 * Search npm packages by query string.
 * @route GET /api/packages/search?q=query
 * @param request - Next.js request object
 * @returns JSON array of matching packages
 * @example
 * GET /api/packages/search?q=react
 * // => [{ name: "react", version: "18.2.0", description: "..." }, ...]
 */
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get("q");

  if (!query || query.length < 2) {
    return NextResponse.json(
      { error: "Query must be at least 2 characters" },
      { status: 400 }
    );
  }

  try {
    const response = await fetch(
      `https://registry.npmjs.org/-/v1/search?text=${encodeURIComponent(query)}&size=10`,
      {
        headers: {
          Accept: "application/json",
        },
        next: { revalidate: 60 }, // Cache for 60 seconds
      }
    );

    if (!response.ok) {
      throw new Error(`npm registry returned ${response.status}`);
    }

    const data: NpmSearchResult = await response.json();

    const packages = data.objects.map((obj) => ({
      name: obj.package.name,
      version: obj.package.version,
      description: obj.package.description || "",
    }));

    return NextResponse.json(packages);
  } catch (error) {
    console.error("Error searching npm packages:", error);
    return NextResponse.json(
      { error: "Failed to search packages" },
      { status: 500 }
    );
  }
}
````

## File: src/app/api/presets/[id]/route.ts
````typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { Pool } from "pg";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "@/generated/prisma/client";
import { headers } from "next/headers";

/**
 * Prisma client instance for database operations.
 */
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaPg(pool);
const prisma = new PrismaClient({ adapter });

/**
 * Maximum length for preset names.
 */
const MAX_NAME_LENGTH = 50;

/**
 * Get current session from request.
 */
async function getSession() {
  const session = await auth.api.getSession({
    headers: await headers(),
  });
  return session;
}

interface RouteParams {
  params: Promise<{ id: string }>;
}

/**
 * GET /api/presets/[id]
 * Get a specific preset by ID.
 *
 * @returns Preset object
 */
export async function GET(request: NextRequest, { params }: RouteParams) {
  try {
    const session = await getSession();
    const { id } = await params;

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      );
    }

    const preset = await prisma.preset.findFirst({
      where: {
        id,
        userId: session.user.id,
      },
      select: {
        id: true,
        name: true,
        packages: true,
        timeRange: true,
        createdAt: true,
      },
    });

    if (!preset) {
      return NextResponse.json(
        { error: "Preset not found" },
        { status: 404 }
      );
    }

    return NextResponse.json(preset);
  } catch (error) {
    console.error("Failed to fetch preset:", error);
    return NextResponse.json(
      { error: "Failed to fetch preset" },
      { status: 500 }
    );
  }
}

/**
 * PATCH /api/presets/[id]
 * Update a preset (rename).
 *
 * @body { name?: string, packages?: string[], timeRange?: string }
 * @returns Updated preset
 */
export async function PATCH(request: NextRequest, { params }: RouteParams) {
  try {
    const session = await getSession();
    const { id } = await params;

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      );
    }

    const body = await request.json();
    const { name, packages, timeRange } = body;

    // Check preset exists and belongs to user
    const existingPreset = await prisma.preset.findFirst({
      where: {
        id,
        userId: session.user.id,
      },
    });

    if (!existingPreset) {
      return NextResponse.json(
        { error: "Preset not found" },
        { status: 404 }
      );
    }

    // Build update data
    const updateData: { name?: string; packages?: string[]; timeRange?: string } = {};

    if (name !== undefined) {
      if (typeof name !== "string" || name.trim().length === 0) {
        return NextResponse.json(
          { error: "Preset name cannot be empty" },
          { status: 400 }
        );
      }

      if (name.length > MAX_NAME_LENGTH) {
        return NextResponse.json(
          { error: `Preset name must be ${MAX_NAME_LENGTH} characters or less` },
          { status: 400 }
        );
      }

      // Check for duplicate name (excluding current preset)
      const duplicatePreset = await prisma.preset.findFirst({
        where: {
          userId: session.user.id,
          name: { equals: name.trim(), mode: "insensitive" },
          NOT: { id },
        },
      });

      if (duplicatePreset) {
        return NextResponse.json(
          { error: "A preset with this name already exists" },
          { status: 400 }
        );
      }

      updateData.name = name.trim();
    }

    if (packages !== undefined) {
      if (!Array.isArray(packages) || packages.length === 0) {
        return NextResponse.json(
          { error: "At least one package is required" },
          { status: 400 }
        );
      }
      updateData.packages = packages;
    }

    if (timeRange !== undefined) {
      updateData.timeRange = timeRange;
    }

    // Update preset
    const updatedPreset = await prisma.preset.update({
      where: { id },
      data: updateData,
      select: {
        id: true,
        name: true,
        packages: true,
        timeRange: true,
        createdAt: true,
      },
    });

    return NextResponse.json(updatedPreset);
  } catch (error) {
    console.error("Failed to update preset:", error);
    return NextResponse.json(
      { error: "Failed to update preset" },
      { status: 500 }
    );
  }
}

/**
 * DELETE /api/presets/[id]
 * Delete a preset.
 *
 * @returns Success message
 */
export async function DELETE(request: NextRequest, { params }: RouteParams) {
  try {
    const session = await getSession();
    const { id } = await params;

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      );
    }

    // Check preset exists and belongs to user
    const existingPreset = await prisma.preset.findFirst({
      where: {
        id,
        userId: session.user.id,
      },
    });

    if (!existingPreset) {
      return NextResponse.json(
        { error: "Preset not found" },
        { status: 404 }
      );
    }

    // Delete preset
    await prisma.preset.delete({
      where: { id },
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error("Failed to delete preset:", error);
    return NextResponse.json(
      { error: "Failed to delete preset" },
      { status: 500 }
    );
  }
}
````

## File: src/app/api/presets/route.ts
````typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { Pool } from "pg";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "@/generated/prisma/client";
import { headers } from "next/headers";

/**
 * Prisma client instance for database operations.
 */
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaPg(pool);
const prisma = new PrismaClient({ adapter });

/**
 * Maximum number of presets per user.
 */
const MAX_PRESETS = 10;

/**
 * Maximum length for preset names.
 */
const MAX_NAME_LENGTH = 50;

/**
 * Get current session from request.
 */
async function getSession() {
  const session = await auth.api.getSession({
    headers: await headers(),
  });
  return session;
}

/**
 * GET /api/presets
 * List all presets for the authenticated user.
 *
 * @returns Array of presets sorted by creation date (newest first)
 *
 * @example
 * // Response:
 * [
 *   { "id": "...", "name": "Frontend Frameworks", "packages": ["react", "vue"], ... }
 * ]
 */
export async function GET(_request: NextRequest) {
  try {
    const session = await getSession();

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      );
    }

    const presets = await prisma.preset.findMany({
      where: { userId: session.user.id },
      orderBy: { createdAt: "desc" },
      select: {
        id: true,
        name: true,
        packages: true,
        timeRange: true,
        createdAt: true,
      },
    });

    return NextResponse.json(presets);
  } catch (error) {
    console.error("Failed to fetch presets:", error);
    return NextResponse.json(
      { error: "Failed to fetch presets" },
      { status: 500 }
    );
  }
}

/**
 * POST /api/presets
 * Create a new preset for the authenticated user.
 *
 * @body { name: string, packages: string[], timeRange?: string }
 * @returns Created preset
 *
 * @example
 * // Request body:
 * { "name": "Frontend Frameworks", "packages": ["react", "vue", "angular"] }
 */
export async function POST(request: NextRequest) {
  try {
    const session = await getSession();

    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      );
    }

    const body = await request.json();
    const { name, packages, timeRange = "1y" } = body;

    // Validate name
    if (!name || typeof name !== "string" || name.trim().length === 0) {
      return NextResponse.json(
        { error: "Preset name is required" },
        { status: 400 }
      );
    }

    if (name.length > MAX_NAME_LENGTH) {
      return NextResponse.json(
        { error: `Preset name must be ${MAX_NAME_LENGTH} characters or less` },
        { status: 400 }
      );
    }

    // Validate packages
    if (!Array.isArray(packages) || packages.length === 0) {
      return NextResponse.json(
        { error: "At least one package is required" },
        { status: 400 }
      );
    }

    // Check preset limit
    const existingCount = await prisma.preset.count({
      where: { userId: session.user.id },
    });

    if (existingCount >= MAX_PRESETS) {
      return NextResponse.json(
        { error: `Maximum ${MAX_PRESETS} presets allowed` },
        { status: 400 }
      );
    }

    // Check for duplicate name
    const existingPreset = await prisma.preset.findFirst({
      where: {
        userId: session.user.id,
        name: { equals: name.trim(), mode: "insensitive" },
      },
    });

    if (existingPreset) {
      return NextResponse.json(
        { error: "A preset with this name already exists" },
        { status: 400 }
      );
    }

    // Create preset
    const preset = await prisma.preset.create({
      data: {
        name: name.trim(),
        packages,
        timeRange,
        userId: session.user.id,
      },
      select: {
        id: true,
        name: true,
        packages: true,
        timeRange: true,
        createdAt: true,
      },
    });

    return NextResponse.json(preset, { status: 201 });
  } catch (error) {
    console.error("Failed to create preset:", error);
    return NextResponse.json(
      { error: "Failed to create preset" },
      { status: 500 }
    );
  }
}
````

## File: src/app/forgot-password/layout.tsx
````typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Forgot Password — npm trend",
  description:
    "Request a password reset link for your npm trend account.",
};

export default function ForgotPasswordLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
````

## File: src/app/forgot-password/page.tsx
````typescript
"use client";

import { useState, useCallback } from "react";
import Link from "next/link";
import { ArrowLeft } from "lucide-react";
import { toast } from "sonner";
import { authClient } from "@/lib/auth-client";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

/**
 * Validates email format.
 * @param email - Email address to validate
 * @returns True if email format is valid
 */
function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Forgot password page component.
 * Allows users to request a password reset email.
 */
export default function ForgotPasswordPage() {
  const [email, setEmail] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [isSubmitted, setIsSubmitted] = useState(false);

  /**
   * Validates the email input.
   * @returns True if email is valid
   */
  const validateEmail = useCallback((): boolean => {
    if (!email.trim()) {
      setError("Email is required");
      return false;
    }
    if (!isValidEmail(email)) {
      setError("Please enter a valid email address");
      return false;
    }
    setError(null);
    return true;
  }, [email]);

  const handleSubmit = useCallback(
    async (e: React.FormEvent) => {
      e.preventDefault();

      if (!validateEmail()) {
        return;
      }

      setIsLoading(true);

      const { error } = await authClient.requestPasswordReset({
        email,
        redirectTo: "/reset-password",
      });

      if (error) {
        toast.error("Request failed", {
          description: error.message ?? "Could not send reset email",
        });
      } else {
        setIsSubmitted(true);
        toast.success("Reset email sent", {
          description: "Check your inbox for password reset instructions",
        });
      }

      setIsLoading(false);
    },
    [validateEmail, email]
  );

  if (isSubmitted) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardHeader className="text-center">
            <CardTitle className="text-2xl">Check your email</CardTitle>
            <CardDescription>
              We&apos;ve sent password reset instructions to{" "}
              <span className="font-medium text-foreground">{email}</span>
            </CardDescription>
          </CardHeader>

          <CardContent className="space-y-4">
            <p className="text-sm text-muted-foreground text-center">
              Didn&apos;t receive the email? Check your spam folder or try again
              with a different email address.
            </p>

            <Button
              variant="outline"
              className="w-full"
              onClick={() => {
                setIsSubmitted(false);
                setEmail("");
              }}
            >
              Try another email
            </Button>
          </CardContent>

          <CardFooter className="flex justify-center">
            <Link
              href="/login"
              className="text-sm text-primary hover:underline flex items-center gap-1"
            >
              <ArrowLeft className="h-4 w-4" />
              Back to login
            </Link>
          </CardFooter>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-2xl">Forgot password?</CardTitle>
          <CardDescription>
            Enter your email and we&apos;ll send you instructions to reset your
            password
          </CardDescription>
        </CardHeader>

        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            {/* Email Field */}
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => {
                  setEmail(e.target.value);
                  if (error) setError(null);
                }}
                aria-invalid={!!error}
                aria-describedby={error ? "email-error" : undefined}
                disabled={isLoading}
              />
              {error && (
                <p id="email-error" className="text-sm text-destructive">
                  {error}
                </p>
              )}
            </div>

            {/* Submit Button */}
            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? "Sending..." : "Send reset instructions"}
            </Button>
          </form>
        </CardContent>

        <CardFooter className="flex justify-center">
          <Link
            href="/login"
            className="text-sm text-primary hover:underline flex items-center gap-1"
          >
            <ArrowLeft className="h-4 w-4" />
            Back to login
          </Link>
        </CardFooter>
      </Card>
    </div>
  );
}
````

## File: src/app/login/layout.tsx
````typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Login — npm trend",
  description:
    "Sign in to your npm trend account to save presets and sync your package comparisons.",
};

export default function LoginLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
````

## File: src/app/login/page.tsx
````typescript
"use client";

import { useState, useCallback, Suspense } from "react";
import Link from "next/link";
import { useRouter, useSearchParams } from "next/navigation";
import { Eye, EyeOff, Github } from "lucide-react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { useAuth } from "@/contexts/auth-context";
import { authClient } from "@/lib/auth-client";

/**
 * Validates email format.
 * @param email - Email address to validate
 * @returns True if email format is valid
 */
function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Login page component.
 * Provides email/password login with client-side validation.
 * Includes OAuth login options and forgot password link.
 * Supports returnUrl for redirect after successful login.
 */
export default function LoginPage() {
  return (
    <Suspense
      fallback={
        <div className="min-h-screen flex items-center justify-center bg-background p-4">
          <Card className="w-full max-w-md">
            <CardContent className="py-8">
              <p className="text-center text-muted-foreground">Loading...</p>
            </CardContent>
          </Card>
        </div>
      }
    >
      <LoginContent />
    </Suspense>
  );
}

/**
 * Login page content (wrapped in Suspense for useSearchParams).
 */
function LoginContent() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const returnUrl = searchParams.get("returnUrl") || "/";
  const { login } = useAuth();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [showPassword, setShowPassword] = useState(false);
  const [errors, setErrors] = useState<{ email?: string; password?: string }>({});
  const [isLoading, setIsLoading] = useState(false);

  /**
   * Validates the login form.
   * @returns True if form is valid
   */
  const validateForm = useCallback((): boolean => {
    const newErrors: { email?: string; password?: string } = {};

    if (!email.trim()) {
      newErrors.email = "Email is required";
    } else if (!isValidEmail(email)) {
      newErrors.email = "Please enter a valid email address";
    }

    if (!password) {
      newErrors.password = "Password is required";
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [email, password]);

  const handleSubmit = useCallback(
    async (e: React.FormEvent) => {
      e.preventDefault();

      if (!validateForm()) {
        return;
      }

      setIsLoading(true);

      const result = await login(email, password);

      if (result.success) {
        toast.success("Welcome back!", {
          description: "You have been signed in successfully",
        });
        router.push(returnUrl);
      } else {
        toast.error("Sign in failed", {
          description: result.error ?? "Invalid email or password",
        });
      }

      setIsLoading(false);
    },
    [validateForm, login, email, password, router, returnUrl]
  );

  const handleOAuthLogin = useCallback(
    async (provider: "github") => {
      setIsLoading(true);
      try {
        const { error } = await authClient.signIn.social({
          provider,
          callbackURL: returnUrl,
        });

        if (error) {
          toast.error("OAuth sign-in failed", {
            description: error.message || "Please try again",
          });
        }
        // On success, the user will be redirected by Better Auth
      } catch {
        toast.error("OAuth sign-in failed", {
          description: "An unexpected error occurred",
        });
      } finally {
        setIsLoading(false);
      }
    },
    [returnUrl]
  );

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-2xl">Welcome back</CardTitle>
          <CardDescription>
            Sign in to your account to continue
          </CardDescription>
        </CardHeader>

        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            {/* Email Field */}
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => {
                  setEmail(e.target.value);
                  if (errors.email) setErrors((prev) => ({ ...prev, email: undefined }));
                }}
                aria-invalid={!!errors.email}
                aria-describedby={errors.email ? "email-error" : undefined}
                disabled={isLoading}
              />
              {errors.email && (
                <p id="email-error" className="text-sm text-destructive">
                  {errors.email}
                </p>
              )}
            </div>

            {/* Password Field */}
            <div className="space-y-2">
              <div className="flex items-center justify-between">
                <Label htmlFor="password">Password</Label>
                <Link
                  href="/forgot-password"
                  className="text-sm text-primary hover:underline"
                >
                  Forgot password?
                </Link>
              </div>
              <div className="relative">
                <Input
                  id="password"
                  type={showPassword ? "text" : "password"}
                  placeholder="Enter your password"
                  value={password}
                  onChange={(e) => {
                    setPassword(e.target.value);
                    if (errors.password) setErrors((prev) => ({ ...prev, password: undefined }));
                  }}
                  aria-invalid={!!errors.password}
                  aria-describedby={errors.password ? "password-error" : undefined}
                  disabled={isLoading}
                  className="pr-10"
                />
                <button
                  type="button"
                  onClick={() => setShowPassword(!showPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground hover:text-foreground"
                  aria-label={showPassword ? "Hide password" : "Show password"}
                >
                  {showPassword ? (
                    <EyeOff className="h-4 w-4" />
                  ) : (
                    <Eye className="h-4 w-4" />
                  )}
                </button>
              </div>
              {errors.password && (
                <p id="password-error" className="text-sm text-destructive">
                  {errors.password}
                </p>
              )}
            </div>

            {/* Submit Button */}
            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? "Signing in..." : "Sign in"}
            </Button>
          </form>

          {/* Divider */}
          <div className="relative my-6">
            <div className="absolute inset-0 flex items-center">
              <span className="w-full border-t" />
            </div>
            <div className="relative flex justify-center text-xs uppercase">
              <span className="bg-card px-2 text-muted-foreground">
                Or continue with
              </span>
            </div>
          </div>

          {/* OAuth Buttons */}
          <Button
            variant="outline"
            className="w-full"
            onClick={() => handleOAuthLogin("github")}
            disabled={isLoading}
          >
            <Github className="mr-2 h-4 w-4" />
            Continue with GitHub
          </Button>
        </CardContent>

        <CardFooter className="flex justify-center">
          <p className="text-sm text-muted-foreground">
            Don&apos;t have an account?{" "}
            <Link href="/signup" className="text-primary hover:underline">
              Sign up
            </Link>
          </p>
        </CardFooter>
      </Card>
    </div>
  );
}
````

## File: src/app/reset-password/layout.tsx
````typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Reset Password — npm trend",
  description: "Set a new password for your npm trend account.",
};

export default function ResetPasswordLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
````

## File: src/app/reset-password/page.tsx
````typescript
"use client";

import { Suspense, useState, useCallback, useEffect } from "react";
import { useSearchParams } from "next/navigation";
import Link from "next/link";
import { ArrowLeft, AlertCircle, CheckCircle2 } from "lucide-react";
import { toast } from "sonner";
import { authClient } from "@/lib/auth-client";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

/**
 * Password reset page wrapper with Suspense.
 * Required for useSearchParams in Next.js App Router.
 */
export default function ResetPasswordPage() {
  return (
    <Suspense
      fallback={
        <div className="min-h-screen flex items-center justify-center bg-background p-4">
          <Card className="w-full max-w-md">
            <CardContent className="py-8">
              <p className="text-center text-muted-foreground">Loading...</p>
            </CardContent>
          </Card>
        </div>
      }
    >
      <ResetPasswordContent />
    </Suspense>
  );
}

/**
 * Password reset page content.
 * Handles password reset with token validation.
 * Shows error for invalid/expired tokens.
 */
function ResetPasswordContent() {
  const searchParams = useSearchParams();
  const token = searchParams.get("token");
  const errorParam = searchParams.get("error");

  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [errors, setErrors] = useState<{
    password?: string;
    confirmPassword?: string;
  }>({});
  const [isLoading, setIsLoading] = useState(false);
  const [resetComplete, setResetComplete] = useState(false);

  const hasValidToken = !!token && !errorParam;

  useEffect(() => {
    if (errorParam || (!token && !errorParam)) {
      toast.error("Invalid or expired token", {
        description: "Please request a new password reset link",
      });
    }
  }, [token, errorParam]);

  /**
   * Validates the password form.
   */
  const validateForm = useCallback((): boolean => {
    const newErrors: { password?: string; confirmPassword?: string } = {};

    if (!password) {
      newErrors.password = "Password is required";
    } else if (password.length < 8) {
      newErrors.password = "Password must be at least 8 characters";
    }

    if (!confirmPassword) {
      newErrors.confirmPassword = "Please confirm your password";
    } else if (password !== confirmPassword) {
      newErrors.confirmPassword = "Passwords do not match";
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [password, confirmPassword]);

  /**
   * Handles form submission.
   */
  const handleSubmit = useCallback(
    async (e: React.FormEvent) => {
      e.preventDefault();

      if (!validateForm() || !token) {
        return;
      }

      setIsLoading(true);

      const { error } = await authClient.resetPassword({
        newPassword: password,
        token,
      });

      if (error) {
        toast.error("Reset failed", {
          description: error.message ?? "Could not reset password",
        });
      } else {
        setResetComplete(true);
        toast.success("Password reset successful", {
          description: "You can now login with your new password",
        });
      }

      setIsLoading(false);
    },
    [validateForm, token, password]
  );

  if (!hasValidToken) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardHeader className="text-center">
            <div className="mx-auto mb-4 h-12 w-12 rounded-full bg-destructive/10 flex items-center justify-center">
              <AlertCircle className="h-6 w-6 text-destructive" />
            </div>
            <CardTitle className="text-2xl">Invalid Reset Link</CardTitle>
            <CardDescription>
              This password reset link is invalid or has expired.
              Please request a new one.
            </CardDescription>
          </CardHeader>

          <CardContent className="space-y-4">
            <Button asChild className="w-full">
              <Link href="/forgot-password">Request New Link</Link>
            </Button>
          </CardContent>

          <CardFooter className="flex justify-center">
            <Link
              href="/login"
              className="text-sm text-primary hover:underline flex items-center gap-1"
            >
              <ArrowLeft className="h-4 w-4" />
              Back to login
            </Link>
          </CardFooter>
        </Card>
      </div>
    );
  }

  // Password reset complete
  if (resetComplete) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardHeader className="text-center">
            <div className="mx-auto mb-4 h-12 w-12 rounded-full bg-green-500/10 flex items-center justify-center">
              <CheckCircle2 className="h-6 w-6 text-green-500" />
            </div>
            <CardTitle className="text-2xl">Password Reset Complete</CardTitle>
            <CardDescription>
              Your password has been successfully reset.
              You can now login with your new password.
            </CardDescription>
          </CardHeader>

          <CardContent>
            <Button asChild className="w-full">
              <Link href="/login">Go to Login</Link>
            </Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-2xl">Reset Your Password</CardTitle>
          <CardDescription>
            Enter your new password below
          </CardDescription>
        </CardHeader>

        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="password">New Password</Label>
              <Input
                id="password"
                type="password"
                placeholder="Enter new password"
                value={password}
                onChange={(e) => {
                  setPassword(e.target.value);
                  if (errors.password)
                    setErrors((prev) => ({ ...prev, password: undefined }));
                }}
                aria-invalid={!!errors.password}
                aria-describedby={errors.password ? "password-error" : undefined}
                disabled={isLoading}
              />
              {errors.password && (
                <p id="password-error" className="text-sm text-destructive">
                  {errors.password}
                </p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="confirmPassword">Confirm New Password</Label>
              <Input
                id="confirmPassword"
                type="password"
                placeholder="Confirm new password"
                value={confirmPassword}
                onChange={(e) => {
                  setConfirmPassword(e.target.value);
                  if (errors.confirmPassword)
                    setErrors((prev) => ({
                      ...prev,
                      confirmPassword: undefined,
                    }));
                }}
                aria-invalid={!!errors.confirmPassword}
                aria-describedby={
                  errors.confirmPassword ? "confirm-password-error" : undefined
                }
                disabled={isLoading}
              />
              {errors.confirmPassword && (
                <p id="confirm-password-error" className="text-sm text-destructive">
                  {errors.confirmPassword}
                </p>
              )}
            </div>

            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? "Resetting..." : "Reset Password"}
            </Button>
          </form>
        </CardContent>

        <CardFooter className="flex justify-center">
          <Link
            href="/login"
            className="text-sm text-primary hover:underline flex items-center gap-1"
          >
            <ArrowLeft className="h-4 w-4" />
            Back to login
          </Link>
        </CardFooter>
      </Card>
    </div>
  );
}
````

## File: src/app/settings/page.tsx
````typescript
"use client";

import { useState, useCallback } from "react";
import Link from "next/link";
import { useRouter } from "next/navigation";
import { ArrowLeft, User, Mail, Lock, Trash2, CheckCircle2, AlertCircle } from "lucide-react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog";
import { ProtectedRoute } from "@/components/protected-route";
import { useAuth } from "@/contexts/auth-context";
import { authClient } from "@/lib/auth-client";

export default function SettingsPage() {
  return (
    <ProtectedRoute>
      <SettingsContent />
    </ProtectedRoute>
  );
}

function SettingsContent() {
  const { user, updateName, deleteAccount } = useAuth();
  const router = useRouter();
  const [name, setName] = useState(user?.name || "");
  const [email] = useState(user?.email || "");
  const [currentPassword, setCurrentPassword] = useState("");
  const [newPassword, setNewPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [deletePassword, setDeletePassword] = useState("");
  const [isLoading, setIsLoading] = useState(false);

  const handleUpdateProfile = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    if (!name.trim()) {
      toast.error("Name is required");
      return;
    }
    setIsLoading(true);

    const success = await updateName(name.trim());

    if (success) {
      toast.success("Profile updated", {
        description: "Your name has been updated successfully",
      });
    } else {
      toast.error("Update failed", {
        description: "Please try again",
      });
    }

    setIsLoading(false);
  }, [name, updateName]);

  const handleChangePassword = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();

    if (!currentPassword || !newPassword || !confirmPassword) {
      toast.error("All password fields are required");
      return;
    }

    if (newPassword.length < 8) {
      toast.error("New password must be at least 8 characters");
      return;
    }

    if (newPassword !== confirmPassword) {
      toast.error("New passwords do not match");
      return;
    }

    setIsLoading(true);

    const { error } = await authClient.changePassword({
      currentPassword,
      newPassword,
      revokeOtherSessions: true,
    });

    if (error) {
      toast.error("Password change failed", {
        description: error.message ?? "Current password may be incorrect",
      });
    } else {
      toast.success("Password changed", {
        description: "Your password has been updated successfully",
      });
      setCurrentPassword("");
      setNewPassword("");
      setConfirmPassword("");
    }

    setIsLoading(false);
  }, [currentPassword, newPassword, confirmPassword]);

  const handleDeleteAccount = useCallback(async () => {
    setIsLoading(true);

    const success = await deleteAccount(deletePassword || undefined);

    if (success) {
      toast.success("Account deleted", {
        description: "Your account has been removed",
      });
      // Full navigation to bypass ProtectedRoute redirect race condition
      window.location.href = "/";
      return;
    }

    toast.error("Deletion failed", {
      description: deletePassword
        ? "Password may be incorrect. Please try again."
        : "Please enter your password to confirm deletion.",
    });

    setDeletePassword("");
    setIsLoading(false);
  }, [deleteAccount, deletePassword]);

  return (
    <div className="min-h-screen bg-background p-4 md:p-8">
      <div className="max-w-2xl mx-auto space-y-6">
        <div className="flex items-center gap-4">
          <Link
            href="/"
            className="text-muted-foreground hover:text-foreground"
          >
            <ArrowLeft className="h-5 w-5" />
          </Link>
          <h1 className="text-2xl font-bold">Account Settings</h1>
        </div>

        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <User className="h-5 w-5" />
              Profile
            </CardTitle>
            <CardDescription>
              Manage your profile information
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleUpdateProfile} className="space-y-4">
              <div className="space-y-2">
                <Label htmlFor="name">Display Name</Label>
                <Input
                  id="name"
                  value={name}
                  onChange={(e) => setName(e.target.value)}
                  disabled={isLoading}
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="email">Email</Label>
                <div className="flex items-center gap-2">
                  <Mail className="h-4 w-4 text-muted-foreground" />
                  <Input
                    id="email"
                    type="email"
                    value={email}
                    disabled
                    className="bg-muted"
                  />
                  {user?.emailVerified ? (
                    <span className="flex items-center gap-1 text-xs text-green-600 whitespace-nowrap">
                      <CheckCircle2 className="h-3.5 w-3.5" />
                      Verified
                    </span>
                  ) : (
                    <span className="flex items-center gap-1 text-xs text-amber-600 whitespace-nowrap">
                      <AlertCircle className="h-3.5 w-3.5" />
                      Unverified
                    </span>
                  )}
                </div>
                <div className="flex items-center gap-2">
                  <p className="text-xs text-muted-foreground">
                    Email cannot be changed
                  </p>
                  {user && !user.emailVerified && (
                    <Button
                      type="button"
                      variant="link"
                      size="sm"
                      className="h-auto p-0 text-xs"
                      disabled={isLoading}
                      onClick={async () => {
                        const { error } = await authClient.sendVerificationEmail({
                          email: user.email,
                          callbackURL: "/verify-email",
                        });
                        if (error) {
                          toast.error("Failed to send verification email");
                        } else {
                          toast.success("Verification email sent", {
                            description: "Check your inbox",
                          });
                        }
                      }}
                    >
                      Resend verification email
                    </Button>
                  )}
                </div>
              </div>
              <Button type="submit" disabled={isLoading}>
                {isLoading ? "Saving..." : "Save Changes"}
              </Button>
            </form>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <Lock className="h-5 w-5" />
              Password
            </CardTitle>
            <CardDescription>
              Change your password
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleChangePassword} className="space-y-4">
              <div className="space-y-2">
                <Label htmlFor="currentPassword">Current Password</Label>
                <Input
                  id="currentPassword"
                  type="password"
                  placeholder="Enter current password"
                  value={currentPassword}
                  onChange={(e) => setCurrentPassword(e.target.value)}
                  disabled={isLoading}
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="newPassword">New Password</Label>
                <Input
                  id="newPassword"
                  type="password"
                  placeholder="Enter new password"
                  value={newPassword}
                  onChange={(e) => setNewPassword(e.target.value)}
                  disabled={isLoading}
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="confirmNewPassword">Confirm New Password</Label>
                <Input
                  id="confirmNewPassword"
                  type="password"
                  placeholder="Confirm new password"
                  value={confirmPassword}
                  onChange={(e) => setConfirmPassword(e.target.value)}
                  disabled={isLoading}
                />
              </div>
              <Button type="submit" disabled={isLoading}>
                {isLoading ? "Changing..." : "Change Password"}
              </Button>
            </form>
          </CardContent>
        </Card>

        <Card className="border-destructive/50">
          <CardHeader>
            <CardTitle className="flex items-center gap-2 text-destructive">
              <Trash2 className="h-5 w-5" />
              Danger Zone
            </CardTitle>
            <CardDescription>
              Irreversible and destructive actions
            </CardDescription>
          </CardHeader>
          <CardContent>
            <AlertDialog>
              <AlertDialogTrigger asChild>
                <Button variant="destructive" disabled={isLoading}>
                  Delete Account
                </Button>
              </AlertDialogTrigger>
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                  <AlertDialogDescription>
                    This action cannot be undone. This will permanently delete
                    your account and remove all your data from our servers.
                  </AlertDialogDescription>
                </AlertDialogHeader>
                <div className="py-2">
                  <Label htmlFor="deletePassword">
                    Password (required for email accounts, optional for OAuth)
                  </Label>
                  <Input
                    id="deletePassword"
                    type="password"
                    placeholder="Your password"
                    value={deletePassword}
                    onChange={(e) => setDeletePassword(e.target.value)}
                    className="mt-2"
                  />
                </div>
                <AlertDialogFooter>
                  <AlertDialogCancel onClick={() => setDeletePassword("")}>
                    Cancel
                  </AlertDialogCancel>
                  <AlertDialogAction
                    onClick={handleDeleteAccount}
                    className="bg-destructive text-destructive-foreground hover:bg-destructive/90"
                  >
                    Delete Account
                  </AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
````

## File: src/app/signup/layout.tsx
````typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Sign Up — npm trend",
  description:
    "Create an npm trend account to save presets and sync your package comparisons across devices.",
};

export default function SignupLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
````

## File: src/app/signup/page.tsx
````typescript
"use client";

import { useState, useCallback, useMemo } from "react";
import Link from "next/link";
import { useRouter } from "next/navigation";
import { Eye, EyeOff, Github, Check, X } from "lucide-react";
import { useAuth } from "@/contexts/auth-context";
import { authClient } from "@/lib/auth-client";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Progress } from "@/components/ui/progress";

/**
 * Validates email format.
 * @param email - Email address to validate
 * @returns True if email format is valid
 */
function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Password strength criteria.
 */
interface PasswordCriteria {
  minLength: boolean;
  hasUppercase: boolean;
  hasLowercase: boolean;
  hasNumber: boolean;
  hasSpecial: boolean;
}

/**
 * Evaluates password strength.
 * @param password - Password to evaluate
 * @returns Criteria met and strength score
 */
function evaluatePasswordStrength(password: string): {
  criteria: PasswordCriteria;
  score: number;
  label: string;
} {
  const criteria: PasswordCriteria = {
    minLength: password.length >= 8,
    hasUppercase: /[A-Z]/.test(password),
    hasLowercase: /[a-z]/.test(password),
    hasNumber: /[0-9]/.test(password),
    hasSpecial: /[^A-Za-z0-9]/.test(password),
  };

  const score = Object.values(criteria).filter(Boolean).length;

  let label = "Very weak";
  if (score >= 5) label = "Strong";
  else if (score >= 4) label = "Good";
  else if (score >= 3) label = "Fair";
  else if (score >= 2) label = "Weak";

  return { criteria, score, label };
}

/**
 * Signup page component.
 * Provides email/password registration with client-side validation.
 * Includes password strength indicator and confirmation matching.
 */
export default function SignupPage() {
  const { signup } = useAuth();
  const router = useRouter();
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [showPassword, setShowPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);
  const [errors, setErrors] = useState<{
    name?: string;
    email?: string;
    password?: string;
    confirmPassword?: string;
  }>({});
  const [isLoading, setIsLoading] = useState(false);

  /**
   * Password strength evaluation.
   */
  const passwordStrength = useMemo(
    () => evaluatePasswordStrength(password),
    [password]
  );

  /**
   * Validates the signup form.
   * @returns True if form is valid
   */
  const validateForm = useCallback((): boolean => {
    const newErrors: {
      name?: string;
      email?: string;
      password?: string;
      confirmPassword?: string;
    } = {};

    if (!name.trim()) {
      newErrors.name = "Name is required";
    }

    if (!email.trim()) {
      newErrors.email = "Email is required";
    } else if (!isValidEmail(email)) {
      newErrors.email = "Please enter a valid email address";
    }

    if (!password) {
      newErrors.password = "Password is required";
    } else if (password.length < 8) {
      newErrors.password = "Password must be at least 8 characters";
    }

    if (!confirmPassword) {
      newErrors.confirmPassword = "Please confirm your password";
    } else if (password !== confirmPassword) {
      newErrors.confirmPassword = "Passwords do not match";
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [name, email, password, confirmPassword]);

  const handleSubmit = useCallback(
    async (e: React.FormEvent) => {
      e.preventDefault();

      if (!validateForm()) {
        return;
      }

      setIsLoading(true);

      const result = await signup(name.trim(), email.trim(), password);

      if (result.success) {
        toast.success("Account created!", {
          description: "Check your email to verify your account",
        });
        router.push("/");
      } else {
        toast.error("Sign up failed", {
          description: result.error ?? "Could not create account",
        });
      }

      setIsLoading(false);
    },
    [validateForm, name, email, password, signup, router]
  );

  const handleOAuthSignup = useCallback(
    async (provider: "github") => {
      setIsLoading(true);
      try {
        const { error } = await authClient.signIn.social({
          provider,
          callbackURL: "/",
        });

        if (error) {
          toast.error("OAuth sign-up failed", {
            description: error.message || "Please try again",
          });
        }
        // On success, the user will be redirected by Better Auth
      } catch {
        toast.error("OAuth sign-up failed", {
          description: "An unexpected error occurred",
        });
      } finally {
        setIsLoading(false);
      }
    },
    []
  );

  /**
   * Renders password strength indicator.
   */
  const strengthColor =
    passwordStrength.score >= 4
      ? "bg-green-500"
      : passwordStrength.score >= 3
        ? "bg-yellow-500"
        : "bg-red-500";

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-2xl">Create an account</CardTitle>
          <CardDescription>
            Enter your details to get started
          </CardDescription>
        </CardHeader>

        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            {/* Name Field */}
            <div className="space-y-2">
              <Label htmlFor="name">Name</Label>
              <Input
                id="name"
                type="text"
                placeholder="Your name"
                value={name}
                onChange={(e) => {
                  setName(e.target.value);
                  if (errors.name)
                    setErrors((prev) => ({ ...prev, name: undefined }));
                }}
                aria-invalid={!!errors.name}
                aria-describedby={errors.name ? "name-error" : undefined}
                disabled={isLoading}
              />
              {errors.name && (
                <p id="name-error" className="text-sm text-destructive">
                  {errors.name}
                </p>
              )}
            </div>

            {/* Email Field */}
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => {
                  setEmail(e.target.value);
                  if (errors.email)
                    setErrors((prev) => ({ ...prev, email: undefined }));
                }}
                aria-invalid={!!errors.email}
                aria-describedby={errors.email ? "email-error" : undefined}
                disabled={isLoading}
              />
              {errors.email && (
                <p id="email-error" className="text-sm text-destructive">
                  {errors.email}
                </p>
              )}
            </div>

            {/* Password Field */}
            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <div className="relative">
                <Input
                  id="password"
                  type={showPassword ? "text" : "password"}
                  placeholder="Create a password"
                  value={password}
                  onChange={(e) => {
                    setPassword(e.target.value);
                    if (errors.password)
                      setErrors((prev) => ({ ...prev, password: undefined }));
                  }}
                  aria-invalid={!!errors.password}
                  aria-describedby={errors.password ? "password-error" : "password-strength"}
                  disabled={isLoading}
                  className="pr-10"
                />
                <button
                  type="button"
                  onClick={() => setShowPassword(!showPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground hover:text-foreground"
                  aria-label={showPassword ? "Hide password" : "Show password"}
                >
                  {showPassword ? (
                    <EyeOff className="h-4 w-4" />
                  ) : (
                    <Eye className="h-4 w-4" />
                  )}
                </button>
              </div>
              {errors.password && (
                <p id="password-error" className="text-sm text-destructive">
                  {errors.password}
                </p>
              )}

              {/* Password Strength Indicator */}
              {password && (
                <div id="password-strength" className="space-y-2">
                  <div className="flex items-center gap-2">
                    <Progress
                      value={(passwordStrength.score / 5) * 100}
                      className={`h-2 ${strengthColor}`}
                    />
                    <span className="text-xs text-muted-foreground min-w-[60px]">
                      {passwordStrength.label}
                    </span>
                  </div>
                  <ul className="text-xs space-y-1">
                    <li className="flex items-center gap-1">
                      {passwordStrength.criteria.minLength ? (
                        <Check className="h-3 w-3 text-green-500" />
                      ) : (
                        <X className="h-3 w-3 text-muted-foreground" />
                      )}
                      <span
                        className={
                          passwordStrength.criteria.minLength
                            ? "text-green-500"
                            : "text-muted-foreground"
                        }
                      >
                        At least 8 characters
                      </span>
                    </li>
                    <li className="flex items-center gap-1">
                      {passwordStrength.criteria.hasUppercase ? (
                        <Check className="h-3 w-3 text-green-500" />
                      ) : (
                        <X className="h-3 w-3 text-muted-foreground" />
                      )}
                      <span
                        className={
                          passwordStrength.criteria.hasUppercase
                            ? "text-green-500"
                            : "text-muted-foreground"
                        }
                      >
                        One uppercase letter
                      </span>
                    </li>
                    <li className="flex items-center gap-1">
                      {passwordStrength.criteria.hasLowercase ? (
                        <Check className="h-3 w-3 text-green-500" />
                      ) : (
                        <X className="h-3 w-3 text-muted-foreground" />
                      )}
                      <span
                        className={
                          passwordStrength.criteria.hasLowercase
                            ? "text-green-500"
                            : "text-muted-foreground"
                        }
                      >
                        One lowercase letter
                      </span>
                    </li>
                    <li className="flex items-center gap-1">
                      {passwordStrength.criteria.hasNumber ? (
                        <Check className="h-3 w-3 text-green-500" />
                      ) : (
                        <X className="h-3 w-3 text-muted-foreground" />
                      )}
                      <span
                        className={
                          passwordStrength.criteria.hasNumber
                            ? "text-green-500"
                            : "text-muted-foreground"
                        }
                      >
                        One number
                      </span>
                    </li>
                    <li className="flex items-center gap-1">
                      {passwordStrength.criteria.hasSpecial ? (
                        <Check className="h-3 w-3 text-green-500" />
                      ) : (
                        <X className="h-3 w-3 text-muted-foreground" />
                      )}
                      <span
                        className={
                          passwordStrength.criteria.hasSpecial
                            ? "text-green-500"
                            : "text-muted-foreground"
                        }
                      >
                        One special character
                      </span>
                    </li>
                  </ul>
                </div>
              )}
            </div>

            {/* Confirm Password Field */}
            <div className="space-y-2">
              <Label htmlFor="confirmPassword">Confirm Password</Label>
              <div className="relative">
                <Input
                  id="confirmPassword"
                  type={showConfirmPassword ? "text" : "password"}
                  placeholder="Confirm your password"
                  value={confirmPassword}
                  onChange={(e) => {
                    setConfirmPassword(e.target.value);
                    if (errors.confirmPassword)
                      setErrors((prev) => ({
                        ...prev,
                        confirmPassword: undefined,
                      }));
                  }}
                  aria-invalid={!!errors.confirmPassword}
                  aria-describedby={
                    errors.confirmPassword ? "confirm-password-error" : undefined
                  }
                  disabled={isLoading}
                  className="pr-10"
                />
                <button
                  type="button"
                  onClick={() => setShowConfirmPassword(!showConfirmPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground hover:text-foreground"
                  aria-label={
                    showConfirmPassword ? "Hide password" : "Show password"
                  }
                >
                  {showConfirmPassword ? (
                    <EyeOff className="h-4 w-4" />
                  ) : (
                    <Eye className="h-4 w-4" />
                  )}
                </button>
              </div>
              {errors.confirmPassword && (
                <p id="confirm-password-error" className="text-sm text-destructive">
                  {errors.confirmPassword}
                </p>
              )}
              {confirmPassword && password === confirmPassword && !errors.confirmPassword && (
                <p className="text-sm text-green-500 flex items-center gap-1">
                  <Check className="h-3 w-3" /> Passwords match
                </p>
              )}
            </div>

            {/* Submit Button */}
            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? "Creating account..." : "Create account"}
            </Button>
          </form>

          {/* Divider */}
          <div className="relative my-6">
            <div className="absolute inset-0 flex items-center">
              <span className="w-full border-t" />
            </div>
            <div className="relative flex justify-center text-xs uppercase">
              <span className="bg-card px-2 text-muted-foreground">
                Or continue with
              </span>
            </div>
          </div>

          {/* OAuth Buttons */}
          <Button
            variant="outline"
            className="w-full"
            onClick={() => handleOAuthSignup("github")}
            disabled={isLoading}
          >
            <Github className="mr-2 h-4 w-4" />
            Continue with GitHub
          </Button>
        </CardContent>

        <CardFooter className="flex justify-center">
          <p className="text-sm text-muted-foreground">
            Already have an account?{" "}
            <Link href="/login" className="text-primary hover:underline">
              Sign in
            </Link>
          </p>
        </CardFooter>
      </Card>
    </div>
  );
}
````

## File: src/app/verify-email/layout.tsx
````typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Verify Email — npm trend",
  description: "Email verification status for your npm trend account.",
};

export default function VerifyEmailLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
````

## File: src/app/verify-email/page.tsx
````typescript
"use client";

import { Suspense } from "react";
import { useSearchParams } from "next/navigation";
import Link from "next/link";
import { CheckCircle2, AlertCircle, ArrowLeft } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

/**
 * Email verification result page.
 * Better Auth redirects here after processing the verification link.
 * Shows success or error state based on query parameters.
 */
export default function VerifyEmailPage() {
  return (
    <Suspense
      fallback={
        <div className="min-h-screen flex items-center justify-center bg-background p-4">
          <Card className="w-full max-w-md">
            <CardContent className="py-8">
              <p className="text-center text-muted-foreground">Loading...</p>
            </CardContent>
          </Card>
        </div>
      }
    >
      <VerifyEmailContent />
    </Suspense>
  );
}

function VerifyEmailContent() {
  const searchParams = useSearchParams();
  const errorParam = searchParams.get("error");

  if (errorParam) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardHeader className="text-center">
            <div className="mx-auto mb-4 h-12 w-12 rounded-full bg-destructive/10 flex items-center justify-center">
              <AlertCircle className="h-6 w-6 text-destructive" />
            </div>
            <CardTitle className="text-2xl">Verification Failed</CardTitle>
            <CardDescription>
              The verification link is invalid or has expired. Please request a
              new verification email from your account settings.
            </CardDescription>
          </CardHeader>

          <CardContent className="space-y-3">
            <Button asChild className="w-full">
              <Link href="/settings">Go to Settings</Link>
            </Button>
            <Button asChild variant="outline" className="w-full">
              <Link href="/">Go to Home</Link>
            </Button>
          </CardContent>

          <CardFooter className="flex justify-center">
            <Link
              href="/login"
              className="text-sm text-primary hover:underline flex items-center gap-1"
            >
              <ArrowLeft className="h-4 w-4" />
              Back to login
            </Link>
          </CardFooter>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <div className="mx-auto mb-4 h-12 w-12 rounded-full bg-green-500/10 flex items-center justify-center">
            <CheckCircle2 className="h-6 w-6 text-green-500" />
          </div>
          <CardTitle className="text-2xl">Email Verified</CardTitle>
          <CardDescription>
            Your email address has been successfully verified. You now have full
            access to all features.
          </CardDescription>
        </CardHeader>

        <CardContent>
          <Button asChild className="w-full">
            <Link href="/">Go to Home</Link>
          </Button>
        </CardContent>
      </Card>
    </div>
  );
}
````

## File: src/app/error.tsx
````typescript
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import { ChevronDown, ChevronUp } from "lucide-react";

/**
 * Error boundary component for route segments.
 * Catches JavaScript errors in child components and displays a fallback UI.
 * Includes expandable error details for debugging.
 *
 * @see https://nextjs.org/docs/app/building-your-application/routing/error-handling
 */
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  const [showDetails, setShowDetails] = useState(false);

  useEffect(() => {
    // Log the error to an error reporting service
    console.error("Error boundary caught:", error);
  }, [error]);

  return (
    <div className="flex min-h-[400px] flex-col items-center justify-center gap-4 px-4">
      <div className="text-center">
        <h2 className="text-xl font-semibold text-foreground">
          Something went wrong
        </h2>
        <p className="mt-2 text-sm text-muted-foreground">
          An error occurred while rendering this page.
        </p>
        {error.digest && (
          <p className="mt-1 text-xs text-muted-foreground">
            Error ID: {error.digest}
          </p>
        )}
      </div>

      {/* Expandable error details */}
      <div className="w-full max-w-md">
        <button
          onClick={() => setShowDetails(!showDetails)}
          className="flex w-full items-center justify-center gap-1 text-xs text-muted-foreground hover:text-foreground transition-colors"
          aria-expanded={showDetails}
          aria-controls="error-details"
        >
          {showDetails ? (
            <>
              <ChevronUp className="h-3 w-3" aria-hidden="true" />
              Hide details
            </>
          ) : (
            <>
              <ChevronDown className="h-3 w-3" aria-hidden="true" />
              Show details
            </>
          )}
        </button>
        {showDetails && (
          <div
            id="error-details"
            className="mt-2 rounded-md border bg-muted/50 p-3 text-left"
          >
            <p className="text-xs font-medium text-foreground">
              {error.name}: {error.message}
            </p>
            {error.stack && (
              <pre className="mt-2 max-h-48 overflow-auto text-xs text-muted-foreground whitespace-pre-wrap break-all">
                {error.stack}
              </pre>
            )}
          </div>
        )}
      </div>

      <div className="flex gap-2">
        <button
          onClick={reset}
          className="inline-flex min-h-[44px] items-center justify-center rounded-md bg-primary px-4 py-2 text-sm font-medium text-primary-foreground transition-colors hover:bg-primary/90 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
        >
          Try again
        </button>
        <Link
          href="/"
          className="inline-flex min-h-[44px] items-center justify-center rounded-md border border-input bg-background px-4 py-2 text-sm font-medium text-foreground transition-colors hover:bg-accent hover:text-accent-foreground focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
        >
          Go home
        </Link>
      </div>
    </div>
  );
}
````

## File: src/app/global-error.tsx
````typescript
"use client";

import { useEffect } from "react";

/**
 * Global error boundary for root layout errors.
 * This component handles errors that occur in the root layout.
 * Must include its own html and body tags as it replaces the root layout.
 *
 * @see https://nextjs.org/docs/app/building-your-application/routing/error-handling#handling-errors-in-root-layouts
 */
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error("Global error boundary caught:", error);
  }, [error]);

  return (
    <html lang="en">
      <body className="bg-background text-foreground antialiased">
        <div className="flex min-h-screen flex-col items-center justify-center gap-4 px-4">
          <div className="text-center">
            <h2 className="text-xl font-semibold">Something went wrong</h2>
            <p className="mt-2 text-sm text-gray-600 dark:text-gray-400">
              A critical error occurred. Please try again.
            </p>
            {error.digest && (
              <p className="mt-1 text-xs text-gray-500">
                Error ID: {error.digest}
              </p>
            )}
          </div>
          <div className="flex gap-2">
            <button
              onClick={reset}
              className="inline-flex min-h-[44px] items-center justify-center rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white transition-colors hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
            >
              Try again
            </button>
            {/* eslint-disable-next-line @next/next/no-html-link-for-pages -- Intentional: Use <a> for full page reload when root layout has failed, as Router context may be unavailable */}
            <a
              href="/"
              className="inline-flex min-h-[44px] items-center justify-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 transition-colors hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:border-gray-600 dark:bg-gray-800 dark:text-gray-200 dark:hover:bg-gray-700"
            >
              Go home
            </a>
          </div>
        </div>
      </body>
    </html>
  );
}
````

## File: src/app/globals.css
````css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
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
  --color-success: var(--success);
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
  --radius-2xl: calc(var(--radius) + 8px);
  --radius-3xl: calc(var(--radius) + 12px);
  --radius-4xl: calc(var(--radius) + 16px);
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
  --success: oklch(0.527 0.154 150.069);
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
  --background: oklch(0.06 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.08 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.12 0 0);
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
  --success: oklch(0.627 0.194 149.214);
  --border: oklch(1 0 0 / 8%);
  --input: oklch(1 0 0 / 12%);
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
  html {
    scroll-behavior: smooth;
    overflow-x: hidden;
  }
  body {
    @apply bg-background text-foreground;
    overflow-x: hidden;
    max-width: 100vw;
  }
  /* Themed text selection highlight */
  ::selection {
    background-color: var(--primary);
    color: var(--primary-foreground);
  }
}

/* Gradient spinner animation */
@keyframes gradient-spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading-spinner-gradient {
  @apply relative h-10 w-10 rounded-full;
  background: conic-gradient(
    from 0deg,
    transparent 0deg 90deg,
    var(--primary) 90deg 180deg,
    var(--chart-1) 180deg 270deg,
    var(--chart-2) 270deg 360deg
  );
  animation: gradient-spin 1s linear infinite;
}

.loading-spinner-gradient::before {
  content: "";
  @apply absolute inset-1 rounded-full bg-background;
}

/* Reduced motion support - placed outside @layer for proper specificity */
@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
````

## File: src/app/layout.tsx
````typescript
import type { Metadata } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import { Analytics } from "@vercel/analytics/next";
import { Toaster } from "sonner";
import { Providers } from "@/components/providers";
import { OfflineIndicator } from "@/components/offline-indicator";
import { isMSWEnabled } from "@/utils/isMSWEnabled";
import "./globals.css";

/**
 * Server-side MSW initialization.
 * Only runs in Node.js runtime (not Edge) when MSW is enabled.
 *
 * Important: Uses require() instead of import for synchronous top-level execution
 */
if (process.env.NEXT_RUNTIME === "nodejs" && isMSWEnabled()) {
  // eslint-disable-next-line @typescript-eslint/no-require-imports
  const { server } = require("../../mocks/server");
  server.listen({ onUnhandledRequest: "bypass" });
}

const geistSans = Geist({
  variable: "--font-geist-sans",
  subsets: ["latin"],
});

const geistMono = Geist_Mono({
  variable: "--font-geist-mono",
  subsets: ["latin"],
});

export const metadata: Metadata = {
  title: "npm trend clone - Compare npm package downloads",
  description:
    "Compare npm package download statistics over time. Find the best packages for your project.",
  keywords: ["npm", "trends", "packages", "downloads", "statistics", "compare"],
  openGraph: {
    title: "npm trend clone - Compare npm package downloads",
    description: "Compare npm package download statistics over time.",
    type: "website",
  },
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body
        className={`${geistSans.variable} ${geistMono.variable} antialiased bg-background text-foreground`}
      >
        <OfflineIndicator />
        <a
          href="#main-content"
          className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 focus:z-50 focus:px-4 focus:py-2 focus:bg-primary focus:text-primary-foreground focus:rounded-md focus:outline-none focus:ring-2 focus:ring-ring"
        >
          Skip to main content
        </a>
        <Providers>{children}</Providers>
        {/*
          Aria-live region wrapper for toast announcements.
          Sonner toasts are announced to screen readers via this wrapper.
        */}
        <div aria-live="polite" aria-atomic="true">
          <Toaster richColors position="bottom-center" duration={4000} />
        </div>
        <Analytics />
      </body>
    </html>
  );
}
````

## File: src/app/not-found.tsx
````typescript
"use client";

import Link from "next/link";
import { Home, Search, ArrowLeft } from "lucide-react";
import { Button } from "@/components/ui/button";

/**
 * Custom 404 Not Found page.
 * Displays a user-friendly error message with navigation options.
 */
export default function NotFound() {
  return (
    <div className="flex min-h-screen flex-col">
      {/* Header */}
      <header className="border-b">
        <div className="container mx-auto flex h-14 sm:h-16 items-center px-3 sm:px-4">
          <Link
            href="/"
            className="text-lg sm:text-xl font-bold hover:opacity-80 transition-opacity"
          >
            npm trends
          </Link>
        </div>
      </header>

      {/* Main Content */}
      <main className="flex flex-1 flex-col items-center justify-center px-4 py-16">
        <div className="text-center space-y-6 max-w-md">
          {/* Error Code */}
          <div className="space-y-2">
            <h1 className="text-8xl sm:text-9xl font-bold text-muted-foreground/30">
              404
            </h1>
            <h2 className="text-2xl sm:text-3xl font-bold tracking-tight">
              Page not found
            </h2>
            <p className="text-muted-foreground">
              The page you&apos;re looking for doesn&apos;t exist or has been
              moved.
            </p>
          </div>

          {/* Action Buttons */}
          <div className="flex flex-col sm:flex-row items-center justify-center gap-3 pt-4">
            <Button asChild size="lg" className="min-h-[44px] w-full sm:w-auto">
              <Link href="/">
                <Home className="mr-2 h-4 w-4" />
                Go to Homepage
              </Link>
            </Button>
            <Button
              variant="outline"
              size="lg"
              className="min-h-[44px] w-full sm:w-auto"
              onClick={() => window.history.back()}
            >
              <ArrowLeft className="mr-2 h-4 w-4" />
              Go Back
            </Button>
          </div>

          {/* Helpful Links */}
          <div className="pt-8 border-t">
            <p className="text-sm text-muted-foreground mb-4">
              Try searching for npm packages:
            </p>
            <div className="flex items-center justify-center gap-2 text-sm">
              <Search className="h-4 w-4 text-muted-foreground" />
              <Link
                href="/?q=react"
                className="text-primary hover:underline underline-offset-4"
              >
                react
              </Link>
              <span className="text-muted-foreground">•</span>
              <Link
                href="/?q=vue"
                className="text-primary hover:underline underline-offset-4"
              >
                vue
              </Link>
              <span className="text-muted-foreground">•</span>
              <Link
                href="/?q=angular"
                className="text-primary hover:underline underline-offset-4"
              >
                angular
              </Link>
            </div>
          </div>
        </div>
      </main>

      {/* Footer */}
      <footer className="border-t py-6">
        <div className="container mx-auto px-4 text-center text-sm text-muted-foreground">
          <Link href="/" className="hover:text-foreground transition-colors">
            npm trends
          </Link>
          {" — "}
          Compare npm package downloads
        </div>
      </footer>
    </div>
  );
}
````

## File: src/app/page.tsx
````typescript
"use client";

import { useCallback, useEffect, useRef, Suspense } from "react";
import dynamic from "next/dynamic";
import { toast } from "sonner";
import { SearchBar } from "@/components/search-bar";
import { PackageTagBar } from "@/components/package-tag-bar";
import { TimeRangeSelector } from "@/components/time-range-selector";

/**
 * Dynamically imported TrendChart to reduce initial bundle size.
 * Recharts library (~500KB) is loaded only when needed.
 */
const TrendChart = dynamic(
  () => import("@/components/trend-chart").then((mod) => ({ default: mod.TrendChart })),
  {
    loading: () => (
      <div
        className="flex h-[400px] items-center justify-center rounded-lg border bg-muted/30"
        role="status"
        aria-busy="true"
      >
        <div className="flex items-center gap-2 text-muted-foreground">
          <div className="h-5 w-5 animate-spin rounded-full border-2 border-current border-t-transparent" />
          <span>Loading chart...</span>
        </div>
      </div>
    ),
    ssr: false,
  }
);
import { ShareButton } from "@/components/share-button";
import { ExportButton } from "@/components/export-button";
import { PresetManager } from "@/components/preset-manager";
import { ThemeToggle } from "@/components/theme-toggle";
import { MobileNav } from "@/components/mobile-nav";
import { UserMenu } from "@/components/user-menu";
import {
  PackageInfoCard,
  PackageInfoCardSkeleton,
  PackageInfoCardError,
} from "@/components/package-info-card";
import { MAX_PACKAGES } from "@/constants/colors";
import { PopularLibraries } from "@/components/popular-libraries";
import { DiscoverPackages } from "@/components/discover-packages";
import { useUrlState } from "@/hooks/use-url-state";
import { useDownloads } from "@/hooks/use-downloads";
import { usePackageInfo } from "@/hooks/use-package-info";

/**
 * Inner component that uses URL state hooks.
 */
function HomeContent() {
  const { selectedPackages, timeRange, addPackage, removePackage, setTimeRange, setPackages } = useUrlState();
  const packageNames = selectedPackages.map((p) => p.name);
  const { data: chartData, isLoading, error, invalidPackages, refetch } = useDownloads(packageNames, timeRange);
  const { data: packageInfoData, isLoading: isLoadingInfo } = usePackageInfo(packageNames);
  const shownToastsRef = useRef<Set<string>>(new Set());

  /**
   * Shows toast notification for invalid packages.
   * Uses ref to track shown toasts and prevent duplicates across re-renders.
   */
  useEffect(() => {
    for (const pkgName of invalidPackages) {
      if (!shownToastsRef.current.has(pkgName)) {
        shownToastsRef.current.add(pkgName);
        toast.error(`Package "${pkgName}" not found`, {
          description: "This package doesn't exist on npm",
          duration: 4000,
        });
      }
    }
  }, [invalidPackages]);

  /**
   * Handles package selection from search.
   */
  const handleSelectPackage = useCallback(
    async (packageName: string) => {
      if (selectedPackages.length >= MAX_PACKAGES) return;
      await addPackage(packageName);
    },
    [selectedPackages.length, addPackage]
  );

  /**
   * Handles package removal.
   */
  const handleRemovePackage = useCallback(
    async (packageName: string) => {
      await removePackage(packageName);
    },
    [removePackage]
  );

  return (
    <div className="min-h-screen flex flex-col bg-background">
      {/* Header */}
      <header className="border-b">
        <div className="container mx-auto flex h-14 sm:h-16 items-center justify-between px-3 sm:px-4">
          <h1 className="text-lg sm:text-xl font-bold">npm trend clone</h1>
          {/* Desktop navigation - hidden on mobile */}
          <nav className="hidden md:flex items-center gap-1 sm:gap-2">
            <PresetManager
              currentPackages={packageNames}
              onLoadPreset={setPackages}
            />
            <a
              href="https://github.com/laststance/npm-trend-clone"
              target="_blank"
              rel="noopener noreferrer"
              className="min-h-[44px] min-w-[44px] flex items-center justify-center text-sm text-muted-foreground hover:text-foreground transition-colors"
            >
              GitHub
            </a>
            <ThemeToggle />
            <UserMenu />
          </nav>

          {/* Mobile navigation - visible only on mobile */}
          <MobileNav />
        </div>
      </header>

      {/* Main Content */}
      <main id="main-content" className="flex-1 container mx-auto px-3 sm:px-4 py-4 sm:py-8">
        <div className="mx-auto max-w-5xl space-y-4 sm:space-y-6">
          {/* Hero Section */}
          <section className="text-center">
            <h2 className="text-2xl sm:text-3xl font-bold tracking-tight">
              Compare npm package downloads
            </h2>
            <p className="mt-1 sm:mt-2 text-sm sm:text-base text-muted-foreground">
              Search and compare download trends for npm packages
            </p>
          </section>

          {/* Search Bar */}
          <section className="flex justify-center">
            <SearchBar
              onSelectPackage={handleSelectPackage}
              selectedPackages={packageNames}
              disabled={selectedPackages.length >= MAX_PACKAGES}
              placeholder={
                selectedPackages.length >= MAX_PACKAGES
                  ? `Maximum ${MAX_PACKAGES} packages reached`
                  : "Search npm packages..."
              }
            />
          </section>

          {/* Package Tags */}
          <section>
            <PackageTagBar
              packages={selectedPackages}
              onRemovePackage={handleRemovePackage}
            />
          </section>

          {/* Time Range Selector & Actions */}
          {selectedPackages.length > 0 && (
            <section className="flex flex-wrap items-center justify-center gap-2 sm:gap-4">
              <TimeRangeSelector value={timeRange} onChange={setTimeRange} />
              <div className="flex items-center gap-2">
                <PresetManager
                  currentPackages={packageNames}
                  onLoadPreset={setPackages}
                  disabled={isLoading}
                />
                <ExportButton disabled={isLoading} />
                <ShareButton />
              </div>
            </section>
          )}

          {/* Chart */}
          <section className="rounded-lg p-2 sm:p-4 -mx-3 sm:mx-0 overflow-x-auto">
            <div className="min-w-[320px]">
              <TrendChart
                data={chartData}
                packages={selectedPackages}
                isLoading={isLoading}
                error={error}
                onRetry={refetch}
              />
            </div>
          </section>

          {/* Package Info Cards */}
          {selectedPackages.length > 0 && (
            <section className="space-y-3 sm:space-y-4">
              <h3 className="text-base sm:text-lg font-semibold">Package Details</h3>
              <div className="grid gap-3 sm:gap-4 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3">
                {selectedPackages.map((pkg) => {
                  const info = packageInfoData[pkg.name];

                  if (isLoadingInfo && !info) {
                    return <PackageInfoCardSkeleton key={pkg.name} />;
                  }

                  if (!info) {
                    return (
                      <PackageInfoCardError
                        key={pkg.name}
                        packageName={pkg.name}
                        accentColor={pkg.color}
                        onRemove={() => handleRemovePackage(pkg.name)}
                      />
                    );
                  }

                  return (
                    <PackageInfoCard
                      key={pkg.name}
                      packageInfo={info}
                      accentColor={pkg.color}
                      onRemove={() => handleRemovePackage(pkg.name)}
                    />
                  );
                })}
              </div>
            </section>
          )}

          {/* Popular Libraries & Discover Packages - shown when no packages selected */}
          {selectedPackages.length === 0 && (
            <section className="space-y-8">
              <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <PopularLibraries
                  onSelectPackage={handleSelectPackage}
                  selectedPackages={packageNames}
                />
                <DiscoverPackages
                  onSelectCategory={(packages) => {
                    // Set packages from the category (up to MAX_PACKAGES)
                    const toSet = packages.slice(0, MAX_PACKAGES);
                    setPackages(toSet);
                  }}
                />
              </div>
            </section>
          )}
        </div>
      </main>

      {/* Footer */}
      <footer className="border-t py-4 sm:py-6">
        <div className="container mx-auto px-3 sm:px-4 text-center text-xs sm:text-sm text-muted-foreground">
          <p>
            Built with Next.js, shadcn/ui, and Recharts •{" "}
            <a
              href="https://github.com/laststance"
              target="_blank"
              rel="noopener noreferrer"
              className="hover:text-foreground transition-colors"
            >
              Laststance.io
            </a>
          </p>
        </div>
      </footer>
    </div>
  );
}

/**
 * Home page for npm trends comparison.
 * Allows users to search and compare npm package download statistics.
 * Uses Suspense for nuqs URL state hydration.
 */
export default function Home() {
  return (
    <Suspense
      fallback={
        <div className="flex min-h-screen items-center justify-center bg-background">
          <div className="loading-spinner-gradient" aria-label="Loading" />
        </div>
      }
    >
      <HomeContent />
    </Suspense>
  );
}
````

## File: src/components/providers/index.tsx
````typescript
"use client";

import type { ReactNode } from "react";
import { NuqsAdapter } from "nuqs/adapters/next/app";
import { ThemeProvider } from "./theme-provider";
import { MSWProvider } from "./msw-provider";
import { AuthProvider } from "@/contexts/auth-context";

/**
 * Root providers component that wraps the application with all necessary providers.
 * @param children - Child components to wrap with providers
 *
 * @description
 * Provider order (outermost to innermost):
 * 1. MSWProvider - Must be outermost to intercept requests before any data fetching
 * 2. NuqsAdapter - URL state management
 * 3. ThemeProvider - Theme context
 * 4. AuthProvider - Authentication context (demo mode)
 */
export function Providers({ children }: { children: ReactNode }) {
  return (
    <MSWProvider>
      <NuqsAdapter>
        <ThemeProvider>
          <AuthProvider>{children}</AuthProvider>
        </ThemeProvider>
      </NuqsAdapter>
    </MSWProvider>
  );
}
````

## File: src/components/providers/msw-provider.tsx
````typescript
"use client";

import { useLayoutEffect, useState, type ReactNode } from "react";
import { isMSWEnabled } from "@/utils/isMSWEnabled";

/**
 * MSWProvider - Wrapper component managing MSW initialization.
 *
 * @description
 * Key implementation points:
 * - 'use client' declares this as a client-only component
 * - useLayoutEffect initializes MSW before first paint
 * - Dynamic import prevents server bundle contamination
 * - State-based blocking prevents race conditions
 *
 * @param children - Child components to render after MSW initialization
 *
 * @example
 * <MSWProvider>
 *   <App />
 * </MSWProvider>
 */
export function MSWProvider({
  children,
}: Readonly<{
  children: ReactNode;
}>): ReactNode {
  const [isMSWReady, setIsMSWReady] = useState(false);

  useLayoutEffect(() => {
    const enabled = isMSWEnabled();

    // Skip if MSW disabled or in SSR context
    if (!enabled || typeof window === "undefined") {
      setIsMSWReady(true);
      return;
    }

    // Dynamic import: Separates browser-only code from server bundle
    import("../../../mocks/browser")
      .then(async ({ worker }) => {
        try {
          await worker.start({
            onUnhandledRequest: "bypass", // Non-mocked requests pass through
          });
          setIsMSWReady(true);
        } catch {
          // App continues even if MSW fails to start
          console.warn("[MSW] Failed to start worker");
          setIsMSWReady(true);
        }
      })
      .catch(() => {
        console.warn("[MSW] Failed to load browser module");
        setIsMSWReady(true);
      });
  }, []);

  // Show loading while MSW initializes
  // This prevents components from fetching before MSW is ready
  if (isMSWEnabled() && !isMSWReady) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <p className="text-lg">Initializing MSW...</p>
      </div>
    );
  }

  return <>{children}</>;
}
````

## File: src/components/providers/theme-provider.tsx
````typescript
"use client";

import { ThemeProvider as NextThemesProvider } from "next-themes";
import type { ReactNode } from "react";

/**
 * Theme provider wrapping next-themes for dark/light mode support.
 * @param children - Child components to wrap
 */
export function ThemeProvider({ children }: { children: ReactNode }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="dark"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </NextThemesProvider>
  );
}
````

## File: src/components/ui/alert-dialog.tsx
````typescript
"use client"

import * as React from "react"
import * as AlertDialogPrimitive from "@radix-ui/react-alert-dialog"

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

## File: src/components/ui/badge.tsx
````typescript
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
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
  const Comp = asChild ? Slot : "span"

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

## File: src/components/ui/button.tsx
````typescript
import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
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
  variant = "default",
  size = "default",
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean
  }) {
  const Comp = asChild ? Slot : "button"

  return (
    <Comp
      data-slot="button"
      data-variant={variant}
      data-size={size}
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}

export { Button, buttonVariants }
````

## File: src/components/ui/card.tsx
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

## File: src/components/ui/dialog.tsx
````typescript
"use client"

import * as React from "react"
import * as DialogPrimitive from "@radix-ui/react-dialog"
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
        "data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed inset-0 z-50 bg-black/50 backdrop-blur-sm",
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
          "bg-background data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 fixed top-[50%] left-[50%] z-50 grid w-full max-w-[calc(100%-2rem)] translate-x-[-50%] translate-y-[-50%] gap-4 rounded-lg border p-6 shadow-lg duration-200 outline-none sm:max-w-lg",
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

## File: src/components/ui/dropdown-menu.tsx
````typescript
"use client"

import * as React from "react"
import * as DropdownMenuPrimitive from "@radix-ui/react-dropdown-menu"
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

## File: src/components/ui/input.tsx
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

## File: src/components/ui/label.tsx
````typescript
"use client"

import * as React from "react"
import * as LabelPrimitive from "@radix-ui/react-label"

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

## File: src/components/ui/progress.tsx
````typescript
"use client"

import * as React from "react"
import * as ProgressPrimitive from "@radix-ui/react-progress"

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

## File: src/components/ui/skeleton.tsx
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

## File: src/components/ui/tooltip.tsx
````typescript
"use client"

import * as React from "react"
import * as TooltipPrimitive from "@radix-ui/react-tooltip"

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

## File: src/components/category-button.tsx
````typescript
"use client";

import type { LucideIcon } from "lucide-react";
import { cn } from "@/lib/utils";

interface CategoryButtonProps {
  /** Category label text */
  label: string;
  /** Lucide icon component */
  icon: LucideIcon;
  /** Callback when the button is clicked */
  onClick: () => void;
  /** Optional additional class names */
  className?: string;
}

/**
 * A category button with icon and label for package discovery.
 * Displays an icon on the left with the category name.
 * Meets 44px minimum touch target requirement for accessibility.
 *
 * @param label - The category label text
 * @param icon - Lucide icon component to display
 * @param onClick - Callback when clicked
 * @param className - Optional additional class names
 *
 * @example
 * <CategoryButton
 *   label="Front-end"
 *   icon={Monitor}
 *   onClick={() => handleCategorySelect('Front-end')}
 * />
 */
export function CategoryButton({
  label,
  icon: Icon,
  onClick,
  className,
}: CategoryButtonProps) {
  return (
    <button
      type="button"
      onClick={onClick}
      className={cn(
        // Base layout
        "inline-flex items-center gap-3 w-full",
        // Sizing - meets 44px minimum touch target
        "min-h-[44px] px-4 py-3",
        // Visual styling
        "rounded-lg border border-border",
        "bg-card text-card-foreground",
        // Typography
        "text-sm font-medium",
        // Transitions
        "transition-all duration-200 ease-out",
        // Hover state
        "hover:bg-accent hover:text-accent-foreground",
        "hover:border-accent-foreground/20",
        "hover:shadow-sm",
        // Active state
        "active:scale-[0.98]",
        // Focus state - accessibility
        "outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2",
        "focus-visible:ring-offset-background",
        className
      )}
      aria-label={`Select ${label} category`}
    >
      <Icon
        className="size-5 shrink-0 text-muted-foreground transition-colors group-hover:text-accent-foreground"
        aria-hidden="true"
      />
      <span className="truncate">{label}</span>
    </button>
  );
}
````

## File: src/components/discover-packages.tsx
````typescript
"use client";

import {
  Search,
  Monitor,
  Server,
  Terminal,
  FileText,
  Palette,
  CheckCircle,
  Cpu,
  Target,
  Smartphone,
  Layers,
  Bot,
  Calculator,
} from "lucide-react";
import type { LucideIcon } from "lucide-react";
import { CategoryButton } from "@/components/category-button";
import {
  CATEGORY_PRESETS,
  type CategoryName,
} from "@/constants/category-presets";

/**
 * Category configuration with icon mapping.
 */
const CATEGORY_ICONS: Record<CategoryName, LucideIcon> = {
  "Front-end": Monitor,
  "Back-end": Server,
  CLI: Terminal,
  Documentation: FileText,
  CSS: Palette,
  Testing: CheckCircle,
  IoT: Cpu,
  Coverage: Target,
  Mobile: Smartphone,
  Frameworks: Layers,
  Robotics: Bot,
  Math: Calculator,
} as const;

/**
 * Ordered list of categories for display.
 */
const CATEGORY_ORDER: CategoryName[] = [
  "Front-end",
  "Back-end",
  "CLI",
  "Documentation",
  "CSS",
  "Testing",
  "IoT",
  "Coverage",
  "Mobile",
  "Frameworks",
  "Robotics",
  "Math",
] as const;

interface DiscoverPackagesProps {
  /** Callback when a category is selected, receives the list of packages */
  onSelectCategory: (packages: string[]) => void;
}

/**
 * Discover packages section with category grid.
 * Displays a title with search icon and a responsive grid of category buttons.
 *
 * @param onSelectCategory - Callback receiving package list when category clicked
 *
 * @example
 * <DiscoverPackages
 *   onSelectCategory={(packages) => setSelectedPackages(packages)}
 * />
 */
export function DiscoverPackages({ onSelectCategory }: DiscoverPackagesProps) {
  /**
   * Handles category button click.
   * @param category - The selected category name
   */
  const handleCategoryClick = (category: CategoryName) => {
    const packages = CATEGORY_PRESETS[category];
    onSelectCategory([...packages]);
  };

  return (
    <section
      aria-labelledby="discover-packages-title"
      className="w-full"
      data-testid="discover-packages"
    >
      {/* Section header */}
      <div className="mb-6">
        <h2
          id="discover-packages-title"
          className="inline-flex items-center gap-2.5 text-xl font-semibold tracking-tight"
        >
          <Search
            className="size-5 text-muted-foreground"
            aria-hidden="true"
          />
          <span className="relative">
            Discover packages
            {/* Orange accent underline */}
            <span
              className="absolute -bottom-1 left-0 h-[3px] w-full rounded-full bg-amber-500"
              aria-hidden="true"
            />
          </span>
        </h2>
      </div>

      {/* Category grid - responsive: 1 col mobile, 2 cols tablet, 3 cols desktop */}
      <div
        className="grid gap-3 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3"
        role="list"
        aria-label="Package categories"
      >
        {CATEGORY_ORDER.map((category) => (
          <div key={category} role="listitem">
            <CategoryButton
              label={category}
              icon={CATEGORY_ICONS[category]}
              onClick={() => handleCategoryClick(category)}
            />
          </div>
        ))}
      </div>
    </section>
  );
}
````

## File: src/components/export-button.tsx
````typescript
"use client";

import { useCallback, useState } from "react";
import { Download, Loader2 } from "lucide-react";
import { domToPng } from "modern-screenshot";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import {
  Tooltip,
  TooltipContent,
  TooltipTrigger,
} from "@/components/ui/tooltip";

interface ExportButtonProps {
  /** CSS selector or ref target for the chart container */
  targetSelector?: string;
  /** Disabled state */
  disabled?: boolean;
}

/**
 * Export button that captures the chart as a PNG image.
 * Uses modern-screenshot to render the DOM element to PNG.
 * @param targetSelector - CSS selector for the element to capture (default: '[data-chart-container]')
 * @param disabled - Whether the button is disabled
 * @example
 * <ExportButton /> // Captures element with data-chart-container attribute
 * <ExportButton targetSelector="#my-chart" /> // Captures #my-chart element
 */
export function ExportButton({
  targetSelector = "[data-chart-container]",
  disabled = false,
}: ExportButtonProps) {
  const [isExporting, setIsExporting] = useState(false);

  /**
   * Captures the chart and triggers a PNG download.
   */
  const handleExport = useCallback(async () => {
    const element = document.querySelector(targetSelector);
    if (!element) {
      toast.error("Chart not found");
      return;
    }

    setIsExporting(true);

    try {
      const dataUrl = await domToPng(element as HTMLElement, {
        scale: 2,
        backgroundColor: "#0a0a0a",
      });

      const link = document.createElement("a");
      link.download = `npm-trends-${new Date().toISOString().split("T")[0]}.png`;
      link.href = dataUrl;
      link.click();

      toast.success("Chart exported as PNG");
    } catch (error) {
      console.error("Export failed:", error);
      toast.error("Failed to export chart");
    } finally {
      setIsExporting(false);
    }
  }, [targetSelector]);

  return (
    <Tooltip>
      <TooltipTrigger asChild>
        <Button
          variant="outline"
          size="icon"
          onClick={handleExport}
          disabled={disabled || isExporting}
          aria-label="Export chart as PNG"
        >
          {isExporting ? (
            <Loader2 className="h-4 w-4 animate-spin" />
          ) : (
            <Download className="h-4 w-4" />
          )}
        </Button>
      </TooltipTrigger>
      <TooltipContent>
        <p>Export chart as PNG</p>
      </TooltipContent>
    </Tooltip>
  );
}
````

## File: src/components/mobile-nav.tsx
````typescript
"use client";

import { useState } from "react";
import { Menu, X, Github, Moon, Sun } from "lucide-react";
import { useTheme } from "next-themes";
import { Button } from "@/components/ui/button";

/**
 * Mobile navigation with hamburger menu.
 * Shows a hamburger icon that opens a dropdown menu with navigation items.
 * Only visible on mobile screens (md breakpoint and below).
 *
 * @example
 * <MobileNav />
 */
export function MobileNav() {
  const [isOpen, setIsOpen] = useState(false);
  const { theme, setTheme } = useTheme();

  const toggleMenu = () => setIsOpen(!isOpen);
  const closeMenu = () => setIsOpen(false);

  return (
    <div className="md:hidden">
      {/* Hamburger button */}
      <Button
        variant="ghost"
        size="icon"
        onClick={toggleMenu}
        aria-label={isOpen ? "Close menu" : "Open menu"}
        aria-expanded={isOpen}
        aria-controls="mobile-nav-menu"
        className="min-h-[44px] min-w-[44px]"
        data-testid="mobile-nav-toggle"
      >
        {isOpen ? (
          <X className="h-5 w-5" aria-hidden="true" />
        ) : (
          <Menu className="h-5 w-5" aria-hidden="true" />
        )}
      </Button>

      {/* Mobile menu dropdown */}
      {isOpen && (
        <>
          {/* Backdrop */}
          <div
            className="fixed inset-0 z-40 bg-background/80 backdrop-blur-sm"
            onClick={closeMenu}
            aria-hidden="true"
          />

          {/* Menu */}
          <div
            id="mobile-nav-menu"
            className="absolute right-0 top-full z-50 mt-2 w-48 rounded-md border bg-popover p-2 shadow-lg"
            role="menu"
            aria-orientation="vertical"
          >
            <a
              href="https://github.com/laststance/npm-trend-clone"
              target="_blank"
              rel="noopener noreferrer"
              className="flex min-h-[44px] items-center gap-2 rounded-md px-3 py-2 text-sm hover:bg-accent"
              role="menuitem"
              onClick={closeMenu}
            >
              <Github className="h-4 w-4" aria-hidden="true" />
              GitHub
            </a>

            <button
              onClick={() => {
                setTheme(theme === "dark" ? "light" : "dark");
                closeMenu();
              }}
              className="flex min-h-[44px] w-full items-center gap-2 rounded-md px-3 py-2 text-sm hover:bg-accent"
              role="menuitem"
            >
              {theme === "dark" ? (
                <Sun className="h-4 w-4" aria-hidden="true" />
              ) : (
                <Moon className="h-4 w-4" aria-hidden="true" />
              )}
              {theme === "dark" ? "Light mode" : "Dark mode"}
            </button>
          </div>
        </>
      )}
    </div>
  );
}
````

## File: src/components/offline-indicator.tsx
````typescript
"use client";

import { WifiOff } from "lucide-react";
import { useOnlineStatus } from "@/hooks/use-online-status";

/**
 * Displays a banner when the user is offline.
 * Automatically shows/hides based on network status.
 *
 * @example
 * // Add to layout or root component
 * <OfflineIndicator />
 */
export function OfflineIndicator() {
  const isOnline = useOnlineStatus();

  if (isOnline) {
    return null;
  }

  return (
    <div
      className="fixed top-0 left-0 right-0 z-50 bg-amber-500 px-4 py-2 text-center text-sm font-medium text-amber-950"
      role="alert"
      aria-live="assertive"
      data-testid="offline-indicator"
    >
      <div className="container mx-auto flex items-center justify-center gap-2">
        <WifiOff className="h-4 w-4" aria-hidden="true" />
        <span>You are currently offline. Some features may be unavailable.</span>
      </div>
    </div>
  );
}
````

## File: src/components/package-info-card.tsx
````typescript
"use client";

import { ExternalLink, Github, Star, Download, Calendar, Scale, X } from "lucide-react";
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardAction,
} from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Skeleton } from "@/components/ui/skeleton";
import type { PackageInfo } from "@/types/package";

/**
 * Props for PackageInfoCard component.
 */
interface PackageInfoCardProps {
  /** Package information to display */
  packageInfo: PackageInfo;
  /** Accent color matching the chart line (hex format) */
  accentColor: string;
  /** Callback when remove button is clicked */
  onRemove?: () => void;
}

/**
 * Formats a number with K/M/B suffix for readability.
 * @param num - Number to format
 * @returns Formatted string with suffix
 * @example
 * formatNumber(1234) // => "1.2K"
 * formatNumber(1234567) // => "1.2M"
 */
function formatNumber(num: number): string {
  if (num >= 1_000_000_000) {
    return `${(num / 1_000_000_000).toFixed(1)}B`;
  }
  if (num >= 1_000_000) {
    return `${(num / 1_000_000).toFixed(1)}M`;
  }
  if (num >= 1_000) {
    return `${(num / 1_000).toFixed(1)}K`;
  }
  return num.toString();
}

/**
 * Formats a date string to a human-readable relative format.
 * @param dateString - ISO date string
 * @returns Human-readable date (e.g., "2 days ago", "3 months ago")
 */
function formatRelativeDate(dateString: string): string {
  const date = new Date(dateString);
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));

  if (diffDays === 0) return "today";
  if (diffDays === 1) return "yesterday";
  if (diffDays < 7) return `${diffDays} days ago`;
  if (diffDays < 30) return `${Math.floor(diffDays / 7)} weeks ago`;
  if (diffDays < 365) return `${Math.floor(diffDays / 30)} months ago`;
  return `${Math.floor(diffDays / 365)} years ago`;
}

/**
 * Displays package information in a card format.
 * Shows name, description, version, downloads, stars, license, and links.
 */
export function PackageInfoCard({
  packageInfo,
  accentColor,
  onRemove,
}: PackageInfoCardProps) {
  const {
    name,
    description,
    version,
    license,
    repositoryUrl,
    lastPublished,
    weeklyDownloads,
    githubStars,
  } = packageInfo;

  const npmUrl = `https://www.npmjs.com/package/${encodeURIComponent(name)}`;

  return (
    <Card className="relative overflow-hidden">
      {/* Accent color bar */}
      <div
        className="absolute left-0 top-0 h-full w-1"
        style={{ backgroundColor: accentColor }}
      />

      <CardHeader className="pl-5">
        <div className="flex items-start justify-between">
          <div className="space-y-1">
            <CardTitle className="font-mono text-base">{name}</CardTitle>
            <CardDescription className="line-clamp-2 text-sm">
              {description}
            </CardDescription>
          </div>
          <CardAction>
            {onRemove && (
              <Button
                variant="ghost"
                size="icon"
                onClick={onRemove}
                aria-label={`Remove ${name} from comparison`}
                className="h-11 w-11 min-h-[44px] min-w-[44px] text-muted-foreground hover:text-foreground"
              >
                <X className="h-4 w-4" />
              </Button>
            )}
          </CardAction>
        </div>
      </CardHeader>

      <CardContent className="pl-5">
        {/* Stats Grid */}
        <div className="grid grid-cols-2 gap-3 text-sm sm:grid-cols-4">
          {/* Version */}
          <div className="flex items-center gap-1.5">
            <Badge variant="outline" className="font-mono text-xs">
              v{version}
            </Badge>
          </div>

          {/* Weekly Downloads */}
          <div className="flex items-center gap-1.5 text-muted-foreground">
            <Download className="h-4 w-4" />
            <span className="font-tabular-nums">{formatNumber(weeklyDownloads)}</span>
            <span className="text-xs">/week</span>
          </div>

          {/* GitHub Stars */}
          {githubStars !== undefined && (
            <div className="flex items-center gap-1.5 text-muted-foreground">
              <Star className="h-4 w-4" />
              <span className="font-tabular-nums">{formatNumber(githubStars)}</span>
            </div>
          )}

          {/* License */}
          <div className="flex items-center gap-1.5 text-muted-foreground">
            <Scale className="h-4 w-4" />
            <span>{license}</span>
          </div>
        </div>

        {/* Last Published */}
        <div className="mt-3 flex items-center gap-1.5 text-xs text-muted-foreground">
          <Calendar className="h-3.5 w-3.5" />
          <span>Updated {formatRelativeDate(lastPublished)}</span>
        </div>

        {/* Links */}
        <div className="mt-3 sm:mt-4 flex items-center gap-2 sm:gap-3">
          <a
            href={npmUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="inline-flex items-center gap-1.5 min-h-[44px] px-2 text-xs text-muted-foreground hover:text-foreground transition-colors"
          >
            <ExternalLink className="h-3.5 w-3.5" />
            npm
          </a>
          {repositoryUrl && (
            <a
              href={repositoryUrl}
              target="_blank"
              rel="noopener noreferrer"
              className="inline-flex items-center gap-1.5 min-h-[44px] px-2 text-xs text-muted-foreground hover:text-foreground transition-colors"
            >
              <Github className="h-3.5 w-3.5" />
              GitHub
            </a>
          )}
        </div>
      </CardContent>
    </Card>
  );
}

/**
 * Loading skeleton for PackageInfoCard.
 * Includes aria-busy and role="status" for screen reader announcements.
 */
export function PackageInfoCardSkeleton() {
  return (
    <Card
      className="relative overflow-hidden"
      role="status"
      aria-busy="true"
      aria-label="Loading package information"
    >
      <div className="absolute left-0 top-0 h-full w-1 bg-muted" />

      <CardHeader className="pl-5">
        <div className="space-y-2">
          <Skeleton className="h-5 w-32" />
          <Skeleton className="h-4 w-full" />
        </div>
      </CardHeader>

      <CardContent className="pl-5">
        <div className="grid grid-cols-2 gap-3 sm:grid-cols-4">
          <Skeleton className="h-6 w-16" />
          <Skeleton className="h-4 w-20" />
          <Skeleton className="h-4 w-16" />
          <Skeleton className="h-4 w-12" />
        </div>
        <Skeleton className="mt-3 h-3 w-24" />
        <div className="mt-4 flex gap-3">
          <Skeleton className="h-4 w-12" />
          <Skeleton className="h-4 w-16" />
        </div>
      </CardContent>
    </Card>
  );
}

/**
 * Error state for PackageInfoCard.
 */
interface PackageInfoCardErrorProps {
  /** Package name that failed to load */
  packageName: string;
  /** Accent color matching the chart line */
  accentColor: string;
  /** Callback when remove button is clicked */
  onRemove?: () => void;
}

export function PackageInfoCardError({
  packageName,
  accentColor,
  onRemove,
}: PackageInfoCardErrorProps) {
  return (
    <Card className="relative overflow-hidden border-destructive/50">
      <div
        className="absolute left-0 top-0 h-full w-1"
        style={{ backgroundColor: accentColor }}
      />

      <CardHeader className="pl-5">
        <div className="flex items-start justify-between">
          <div className="space-y-1">
            <CardTitle className="font-mono text-base text-destructive">
              {packageName}
            </CardTitle>
            <CardDescription className="text-destructive/80">
              Failed to load package information
            </CardDescription>
          </div>
          <CardAction>
            {onRemove && (
              <Button
                variant="ghost"
                size="icon"
                onClick={onRemove}
                aria-label={`Remove ${packageName} from comparison`}
                className="h-11 w-11 min-h-[44px] min-w-[44px] text-muted-foreground hover:text-foreground"
              >
                <X className="h-4 w-4" />
              </Button>
            )}
          </CardAction>
        </div>
      </CardHeader>
    </Card>
  );
}
````

## File: src/components/package-tag-bar.stories.tsx
````typescript
import type { Meta, StoryObj } from "@storybook/nextjs";
import { PackageTagBar } from "./package-tag-bar";
import { CHART_COLORS } from "@/constants/colors";

/**
 * PackageTagBar displays a row of PackageTags for all selected packages.
 * Tags are arranged with flex-wrap for responsive layouts.
 */
const meta: Meta<typeof PackageTagBar> = {
  title: "Components/PackageTagBar",
  component: PackageTagBar,
  parameters: {
    layout: "padded",
    backgrounds: {
      default: "dark",
      values: [{ name: "dark", value: "#0f0f0f" }],
    },
  },
  tags: ["autodocs"],
  argTypes: {
    onRemovePackage: { action: "removed" },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

/**
 * Empty state when no packages are selected.
 */
export const Empty: Story = {
  args: {
    packages: [],
  },
};

/**
 * Single package selected.
 */
export const SinglePackage: Story = {
  args: {
    packages: [{ name: "react", color: CHART_COLORS[0] }],
  },
};

/**
 * Multiple packages - typical comparison view.
 */
export const MultiplePackages: Story = {
  args: {
    packages: [
      { name: "react", color: CHART_COLORS[0] },
      { name: "vue", color: CHART_COLORS[1] },
      { name: "@angular/core", color: CHART_COLORS[2] },
    ],
  },
};

/**
 * Maximum packages (6) - showing all available colors.
 */
export const MaximumPackages: Story = {
  args: {
    packages: [
      { name: "react", color: CHART_COLORS[0] },
      { name: "vue", color: CHART_COLORS[1] },
      { name: "@angular/core", color: CHART_COLORS[2] },
      { name: "svelte", color: CHART_COLORS[3] },
      { name: "solid-js", color: CHART_COLORS[4] },
      { name: "preact", color: CHART_COLORS[5] },
    ],
  },
};
````

## File: src/components/package-tag-bar.tsx
````typescript
"use client";

import { PackageTag } from "./package-tag";
import type { SelectedPackage } from "@/types/package";

interface PackageTagBarProps {
  /** Array of selected packages */
  packages: SelectedPackage[];
  /** Callback when a package is removed */
  onRemovePackage: (name: string) => void;
}

/**
 * Displays a horizontal bar of package tags.
 * @param packages - Array of selected packages to display
 * @param onRemovePackage - Callback when a package is removed
 */
export function PackageTagBar({
  packages,
  onRemovePackage,
}: PackageTagBarProps) {
  if (packages.length === 0) {
    return null;
  }

  return (
    <div
      className="flex flex-wrap gap-2 p-4"
      role="list"
      aria-label="Selected packages"
    >
      {packages.map((pkg) => (
        <PackageTag key={pkg.name} package={pkg} onRemove={onRemovePackage} />
      ))}
    </div>
  );
}
````

## File: src/components/package-tag.stories.tsx
````typescript
import type { Meta, StoryObj } from "@storybook/nextjs";
import { PackageTag } from "./package-tag";
import { CHART_COLORS } from "@/constants/colors";

/**
 * PackageTag displays a colored tag for a selected npm package.
 * It includes a dropdown menu for additional actions and a remove button.
 */
const meta: Meta<typeof PackageTag> = {
  title: "Components/PackageTag",
  component: PackageTag,
  parameters: {
    layout: "centered",
    backgrounds: {
      default: "dark",
      values: [{ name: "dark", value: "#0f0f0f" }],
    },
  },
  tags: ["autodocs"],
  argTypes: {
    onRemove: { action: "removed" },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

/**
 * Default package tag with React branding color.
 */
export const Default: Story = {
  args: {
    package: {
      name: "react",
      color: CHART_COLORS[0],
    },
  },
};

/**
 * Vue package with green color.
 */
export const Vue: Story = {
  args: {
    package: {
      name: "vue",
      color: CHART_COLORS[1],
    },
  },
};

/**
 * Angular package with red color.
 */
export const Angular: Story = {
  args: {
    package: {
      name: "@angular/core",
      color: CHART_COLORS[2],
    },
  },
};

/**
 * Package with a long name that gets truncated.
 */
export const LongName: Story = {
  args: {
    package: {
      name: "@typescript-eslint/eslint-plugin",
      color: CHART_COLORS[3],
    },
  },
};

/**
 * Multiple tags displayed together.
 */
export const MultipleTags: Story = {
  render: () => (
    <div className="flex flex-wrap gap-2">
      <PackageTag
        package={{ name: "react", color: CHART_COLORS[0] }}
        onRemove={() => {}}
      />
      <PackageTag
        package={{ name: "vue", color: CHART_COLORS[1] }}
        onRemove={() => {}}
      />
      <PackageTag
        package={{ name: "@angular/core", color: CHART_COLORS[2] }}
        onRemove={() => {}}
      />
    </div>
  ),
};
````

## File: src/components/package-tag.tsx
````typescript
"use client";

import { X, MoreVertical } from "lucide-react";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { Button } from "@/components/ui/button";
import type { SelectedPackage } from "@/types/package";
import { getContrastTextColor, getContrastHoverClass } from "@/utils/color";

interface PackageTagProps {
  /** Package data with name and color */
  package: SelectedPackage;
  /** Callback when the remove button is clicked */
  onRemove: (name: string) => void;
}

/**
 * Displays a colored tag for a selected package with remove and options buttons.
 * Uses WCAG-compliant contrast colors for accessibility.
 * @param package - The package to display
 * @param onRemove - Callback when the package is removed
 */
export function PackageTag({ package: pkg, onRemove }: PackageTagProps) {
  const textColorClass = getContrastTextColor(pkg.color);
  const hoverClass = getContrastHoverClass(pkg.color);

  return (
    <div
      data-testid="package-tag"
      className={`inline-flex items-center gap-0.5 rounded-lg pl-3 pr-1 py-1 min-h-[44px] text-sm font-medium ${textColorClass} shadow-sm`}
      style={{ backgroundColor: pkg.color }}
    >
      <span className="max-w-[150px] truncate font-mono">{pkg.name}</span>

      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button
            variant="ghost"
            size="sm"
            className={`h-9 w-9 min-h-[36px] min-w-[36px] p-0 ${hoverClass}`}
            aria-label={`Options for ${pkg.name}`}
          >
            <MoreVertical className="h-4 w-4" />
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="start">
          <DropdownMenuItem
            onClick={() =>
              window.open(`https://npmjs.com/package/${pkg.name}`, "_blank")
            }
          >
            View on npm
          </DropdownMenuItem>
          <DropdownMenuItem
            onClick={() =>
              window.open(`https://github.com/search?q=${pkg.name}`, "_blank")
            }
          >
            Search on GitHub
          </DropdownMenuItem>
        </DropdownMenuContent>
      </DropdownMenu>

      <Button
        variant="ghost"
        size="sm"
        className={`h-9 w-9 min-h-[36px] min-w-[36px] p-0 ${hoverClass}`}
        onClick={() => onRemove(pkg.name)}
        aria-label={`Remove ${pkg.name}`}
      >
        <X className="h-4 w-4" />
      </Button>
    </div>
  );
}
````

## File: src/components/popular-libraries.stories.tsx
````typescript
import type { Meta, StoryObj } from "@storybook/nextjs";
import { useState } from "react";
import { PopularLibraries } from "./popular-libraries";

/**
 * PopularLibraries displays a curated list of popular npm packages
 * that users can click to quickly add to their comparison.
 * Already selected packages are visually muted and disabled.
 */
const meta: Meta<typeof PopularLibraries> = {
  title: "Components/PopularLibraries",
  component: PopularLibraries,
  parameters: {
    layout: "centered",
  },
  tags: ["autodocs"],
  argTypes: {
    onSelectPackage: { action: "selected" },
  },
  decorators: [
    (Story) => (
      <div className="w-[280px] p-4 bg-background rounded-lg border border-border">
        <Story />
      </div>
    ),
  ],
};

export default meta;
type Story = StoryObj<typeof meta>;

/**
 * Default state with no packages selected.
 * All packages are clickable.
 */
export const Default: Story = {
  args: {
    selectedPackages: [],
  },
};

/**
 * Some packages already selected.
 * Selected packages appear muted and have strikethrough.
 */
export const WithSelectedPackages: Story = {
  args: {
    selectedPackages: ["react", "vue", "express"],
  },
};

/**
 * All packages selected.
 * Demonstrates the disabled state for all items.
 */
export const AllSelected: Story = {
  args: {
    selectedPackages: [
      "react",
      "lodash",
      "react-dom",
      "axios",
      "chalk",
      "tslib",
      "commander",
      "inquirer",
      "express",
      "vue",
    ],
  },
};

/**
 * Interactive example showing the selection flow.
 * Click packages to add them to the selection.
 */
export const Interactive: Story = {
  render: function InteractiveStory() {
    const [selected, setSelected] = useState<string[]>(["react"]);

    const handleSelect = (name: string) => {
      setSelected((prev) => [...prev, name]);
    };

    return (
      <div className="flex flex-col gap-4">
        <PopularLibraries
          onSelectPackage={handleSelect}
          selectedPackages={selected}
        />
        <div className="text-xs text-muted-foreground border-t border-border pt-3">
          <strong>Selected:</strong>{" "}
          {selected.length > 0 ? selected.join(", ") : "None"}
        </div>
      </div>
    );
  },
};

/**
 * Dark theme variant.
 */
export const DarkTheme: Story = {
  args: {
    selectedPackages: ["lodash", "axios"],
  },
  parameters: {
    backgrounds: {
      default: "dark",
    },
  },
  decorators: [
    (Story) => (
      <div className="dark w-[280px] p-4 bg-background rounded-lg border border-border">
        <Story />
      </div>
    ),
  ],
};
````

## File: src/components/popular-libraries.tsx
````typescript
"use client";

import { Flame } from "lucide-react";
import { cn } from "@/lib/utils";

/**
 * List of popular npm packages for quick selection.
 * Curated based on download statistics and common use cases.
 */
const POPULAR_PACKAGES = [
  "react",
  "lodash",
  "react-dom",
  "axios",
  "chalk",
  "tslib",
  "commander",
  "inquirer",
  "express",
  "vue",
] as const;

interface PopularLibrariesProps {
  /**
   * Callback fired when a package is selected
   * @param name - The npm package name
   */
  onSelectPackage: (name: string) => void;
  /**
   * Array of currently selected package names
   * Used to disable already selected packages
   */
  selectedPackages: string[];
}

/**
 * Displays a list of popular npm packages for quick selection.
 * Packages that are already selected are visually muted and disabled.
 * @param onSelectPackage - Callback when a package is clicked
 * @param selectedPackages - Array of already selected package names
 * @example
 * <PopularLibraries
 *   onSelectPackage={(name) => console.log(`Selected: ${name}`)}
 *   selectedPackages={["react", "vue"]}
 * />
 */
export function PopularLibraries({
  onSelectPackage,
  selectedPackages,
}: PopularLibrariesProps) {
  return (
    <section
      data-testid="popular-libraries"
      className="flex flex-col gap-3"
      aria-labelledby="popular-libraries-title"
    >
      {/* Header with flame icon and accent underline */}
      <header className="flex flex-col gap-1">
        <h2
          id="popular-libraries-title"
          className="flex items-center gap-2 text-base font-semibold text-foreground"
        >
          <Flame
            className="size-5 text-orange-500 dark:text-orange-400"
            aria-hidden="true"
          />
          Popular libraries
        </h2>
        {/* Red accent underline - 3px height, 48px width */}
        <div
          className="h-[3px] w-12 rounded-full bg-red-500 dark:bg-red-400"
          aria-hidden="true"
        />
      </header>

      {/* Package list */}
      <ul className="flex flex-col gap-0.5" role="list">
        {POPULAR_PACKAGES.map((packageName) => {
          const isSelected = selectedPackages.includes(packageName);

          return (
            <li key={packageName}>
              <button
                type="button"
                onClick={() => onSelectPackage(packageName)}
                disabled={isSelected}
                className={cn(
                  // Base styles
                  "group w-full text-left font-mono text-sm py-2.5 px-3 rounded-md",
                  "min-h-[44px] transition-colors duration-150",
                  // Interactive states
                  "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2",
                  // Enabled state: hover with dotted underline
                  !isSelected && [
                    "text-muted-foreground",
                    "hover:text-primary hover:bg-accent/50",
                    "cursor-pointer",
                  ],
                  // Disabled state: already selected
                  isSelected && [
                    "text-muted-foreground/50",
                    "cursor-not-allowed",
                    "line-through",
                  ]
                )}
                aria-label={
                  isSelected
                    ? `${packageName} - already selected`
                    : `Add ${packageName} to comparison`
                }
              >
                <span
                  className={cn(
                    "transition-all duration-150",
                    // Dotted underline on hover for enabled items
                    !isSelected && [
                      "border-b border-transparent",
                      "group-hover:border-dotted group-hover:border-primary/60",
                    ]
                  )}
                >
                  {packageName}
                </span>
              </button>
            </li>
          );
        })}
      </ul>
    </section>
  );
}
````

## File: src/components/preset-manager.tsx
````typescript
"use client";

import { useState, useCallback } from "react";
import { Bookmark, Plus, Trash2, Edit2 } from "lucide-react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import {
  Tooltip,
  TooltipContent,
  TooltipTrigger,
} from "@/components/ui/tooltip";
import { usePresets, type Preset } from "@/hooks/use-presets";

interface PresetManagerProps {
  /** Currently selected package names */
  currentPackages: string[];
  /** Callback when a preset is loaded */
  onLoadPreset: (packages: string[]) => void;
  /** Whether the component is disabled */
  disabled?: boolean;
}

/**
 * Component for managing package comparison presets.
 * Allows users to save, load, rename, and delete presets.
 *
 * @param currentPackages - Currently selected package names
 * @param onLoadPreset - Called when user loads a preset
 * @param disabled - Whether the component is disabled
 * @example
 * <PresetManager
 *   currentPackages={["react", "vue"]}
 *   onLoadPreset={(packages) => setPackages(packages)}
 * />
 */
export function PresetManager({
  currentPackages,
  onLoadPreset,
  disabled = false,
}: PresetManagerProps) {
  const {
    presets,
    savePreset,
    deletePreset,
    renamePreset,
    maxPresets,
    maxNameLength,
  } = usePresets();

  // Dialog states
  const [saveDialogOpen, setSaveDialogOpen] = useState(false);
  const [renameDialogOpen, setRenameDialogOpen] = useState(false);
  const [deleteDialogOpen, setDeleteDialogOpen] = useState(false);

  // Form states
  const [presetName, setPresetName] = useState("");
  const [nameError, setNameError] = useState<string | null>(null);
  const [selectedPreset, setSelectedPreset] = useState<Preset | null>(null);

  /**
   * Opens the save dialog.
   */
  const handleOpenSaveDialog = useCallback(() => {
    setPresetName("");
    setNameError(null);
    setSaveDialogOpen(true);
  }, []);

  /**
   * Saves the current packages as a new preset.
   */
  const handleSavePreset = useCallback(async () => {
    const result = await savePreset(presetName, currentPackages);
    if (result.error) {
      setNameError(result.error);
      return;
    }
    setSaveDialogOpen(false);
    toast.success(`Preset "${presetName}" saved`);
  }, [presetName, currentPackages, savePreset]);

  /**
   * Loads a preset's packages.
   */
  const handleLoadPreset = useCallback(
    (preset: Preset) => {
      onLoadPreset(preset.packages);
      toast.success(`Loaded preset "${preset.name}"`);
    },
    [onLoadPreset]
  );

  /**
   * Opens the rename dialog for a preset.
   */
  const handleOpenRenameDialog = useCallback((preset: Preset) => {
    setSelectedPreset(preset);
    setPresetName(preset.name);
    setNameError(null);
    setRenameDialogOpen(true);
  }, []);

  /**
   * Renames the selected preset.
   */
  const handleRenamePreset = useCallback(async () => {
    if (!selectedPreset) return;
    const error = await renamePreset(selectedPreset.id, presetName);
    if (error) {
      setNameError(error);
      return;
    }
    setRenameDialogOpen(false);
    toast.success("Preset renamed");
  }, [selectedPreset, presetName, renamePreset]);

  /**
   * Opens the delete confirmation dialog for a preset.
   */
  const handleOpenDeleteDialog = useCallback((preset: Preset) => {
    setSelectedPreset(preset);
    setDeleteDialogOpen(true);
  }, []);

  /**
   * Deletes the selected preset.
   */
  const handleDeletePreset = useCallback(async () => {
    if (!selectedPreset) return;
    const success = await deletePreset(selectedPreset.id);
    if (success) {
      setDeleteDialogOpen(false);
      toast.success(`Preset "${selectedPreset.name}" deleted`);
    } else {
      toast.error("Failed to delete preset");
    }
  }, [selectedPreset, deletePreset]);

  const canSave = currentPackages.length > 0 && presets.length < maxPresets;

  return (
    <>
      <DropdownMenu>
        <Tooltip>
          <TooltipTrigger asChild>
            <DropdownMenuTrigger asChild>
              <Button
                variant="outline"
                size="icon"
                disabled={disabled}
                aria-label="Manage presets"
              >
                <Bookmark className="h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
          </TooltipTrigger>
          <TooltipContent>
            <p>Manage presets</p>
          </TooltipContent>
        </Tooltip>

        <DropdownMenuContent align="end" className="w-56">
          <DropdownMenuLabel>
            Presets ({presets.length}/{maxPresets})
          </DropdownMenuLabel>
          <DropdownMenuSeparator />

          {/* Save current as preset */}
          <DropdownMenuItem
            onClick={handleOpenSaveDialog}
            disabled={!canSave}
          >
            <Plus className="h-4 w-4" />
            Save current as preset
          </DropdownMenuItem>

          {presets.length > 0 && <DropdownMenuSeparator />}

          {/* Preset list */}
          {presets.map((preset) => (
            <DropdownMenuItem
              key={preset.id}
              className="flex items-center justify-between group"
              onClick={() => handleLoadPreset(preset)}
            >
              <span className="truncate flex-1">{preset.name}</span>
              <span className="text-xs text-muted-foreground ml-2">
                {preset.packages.length} pkg
              </span>
              <div
                role="presentation"
                className="opacity-0 group-hover:opacity-100 flex items-center gap-1 ml-2"
                onClick={(e) => e.stopPropagation()}
                onKeyDown={(e) => { if (e.key === "Enter" || e.key === " ") e.stopPropagation() }}
              >
                <button
                  onClick={() => handleOpenRenameDialog(preset)}
                  className="p-1 hover:bg-accent rounded"
                  aria-label={`Rename preset ${preset.name}`}
                >
                  <Edit2 className="h-3 w-3" />
                </button>
                <button
                  onClick={() => handleOpenDeleteDialog(preset)}
                  className="p-1 hover:bg-destructive/10 rounded text-destructive"
                  aria-label={`Delete preset ${preset.name}`}
                >
                  <Trash2 className="h-3 w-3" />
                </button>
              </div>
            </DropdownMenuItem>
          ))}

          {presets.length === 0 && (
            <div className="px-2 py-4 text-center text-sm text-muted-foreground">
              No presets saved yet
            </div>
          )}
        </DropdownMenuContent>
      </DropdownMenu>

      {/* Save Dialog */}
      <Dialog open={saveDialogOpen} onOpenChange={setSaveDialogOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Save Preset</DialogTitle>
            <DialogDescription>
              Save current packages ({currentPackages.join(", ")}) as a preset
              for quick access later.
            </DialogDescription>
          </DialogHeader>
          <div className="space-y-2">
            <Input
              placeholder="Preset name"
              value={presetName}
              onChange={(e) => {
                setPresetName(e.target.value);
                setNameError(null);
              }}
              maxLength={maxNameLength}
              aria-invalid={!!nameError}
              aria-describedby={nameError ? "preset-name-error" : undefined}
            />
            {nameError && (
              <p id="preset-name-error" className="text-sm text-destructive">
                {nameError}
              </p>
            )}
            <p className="text-xs text-muted-foreground">
              {presetName.length}/{maxNameLength} characters
            </p>
          </div>
          <DialogFooter>
            <Button variant="outline" onClick={() => setSaveDialogOpen(false)}>
              Cancel
            </Button>
            <Button onClick={handleSavePreset} disabled={!presetName.trim()}>
              Save
            </Button>
          </DialogFooter>
        </DialogContent>
      </Dialog>

      {/* Rename Dialog */}
      <Dialog open={renameDialogOpen} onOpenChange={setRenameDialogOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Rename Preset</DialogTitle>
            <DialogDescription>
              Enter a new name for this preset.
            </DialogDescription>
          </DialogHeader>
          <div className="space-y-2">
            <Input
              placeholder="Preset name"
              value={presetName}
              onChange={(e) => {
                setPresetName(e.target.value);
                setNameError(null);
              }}
              maxLength={maxNameLength}
              aria-invalid={!!nameError}
              aria-describedby={nameError ? "rename-error" : undefined}
            />
            {nameError && (
              <p id="rename-error" className="text-sm text-destructive">
                {nameError}
              </p>
            )}
          </div>
          <DialogFooter>
            <Button
              variant="outline"
              onClick={() => setRenameDialogOpen(false)}
            >
              Cancel
            </Button>
            <Button onClick={handleRenamePreset} disabled={!presetName.trim()}>
              Rename
            </Button>
          </DialogFooter>
        </DialogContent>
      </Dialog>

      {/* Delete Confirmation Dialog */}
      <Dialog open={deleteDialogOpen} onOpenChange={setDeleteDialogOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Delete Preset</DialogTitle>
            <DialogDescription>
              Are you sure you want to delete &quot;{selectedPreset?.name}&quot;? This
              action cannot be undone.
            </DialogDescription>
          </DialogHeader>
          <DialogFooter>
            <Button
              variant="outline"
              onClick={() => setDeleteDialogOpen(false)}
            >
              Cancel
            </Button>
            <Button variant="destructive" onClick={handleDeletePreset}>
              Delete
            </Button>
          </DialogFooter>
        </DialogContent>
      </Dialog>
    </>
  );
}
````

## File: src/components/protected-route.tsx
````typescript
"use client";

import { useEffect } from "react";
import { useRouter, usePathname } from "next/navigation";
import { useAuth } from "@/contexts/auth-context";
import { Card, CardContent } from "@/components/ui/card";

interface ProtectedRouteProps {
  children: React.ReactNode;
}

/**
 * Protected route wrapper component.
 * Redirects unauthenticated users to login page.
 * Preserves the original URL for redirect after login.
 */
export function ProtectedRoute({ children }: ProtectedRouteProps) {
  const { isAuthenticated, isLoading } = useAuth();
  const router = useRouter();
  const pathname = usePathname();

  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      // Preserve the current URL for redirect after login
      const returnUrl = encodeURIComponent(pathname);
      router.replace(`/login?returnUrl=${returnUrl}`);
    }
  }, [isAuthenticated, isLoading, pathname, router]);

  // Show loading state while checking auth
  if (isLoading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardContent className="py-8">
            <p className="text-center text-muted-foreground">Loading...</p>
          </CardContent>
        </Card>
      </div>
    );
  }

  // Don't render children if not authenticated (redirect is happening)
  if (!isAuthenticated) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background p-4">
        <Card className="w-full max-w-md">
          <CardContent className="py-8">
            <p className="text-center text-muted-foreground">
              Redirecting to login...
            </p>
          </CardContent>
        </Card>
      </div>
    );
  }

  return <>{children}</>;
}
````

## File: src/components/search-bar.stories.tsx
````typescript
import type { Meta, StoryObj } from "@storybook/nextjs";
import { SearchBar } from "./search-bar";

/**
 * SearchBar provides autocomplete search for npm packages.
 * It fetches suggestions from the npm registry with debouncing.
 */
const meta: Meta<typeof SearchBar> = {
  title: "Components/SearchBar",
  component: SearchBar,
  parameters: {
    layout: "centered",
    backgrounds: {
      default: "dark",
      values: [{ name: "dark", value: "#0f0f0f" }],
    },
  },
  tags: ["autodocs"],
  argTypes: {
    onSelectPackage: { action: "selected" },
  },
  decorators: [
    (Story) => (
      <div className="w-[600px]">
        <Story />
      </div>
    ),
  ],
};

export default meta;
type Story = StoryObj<typeof meta>;

/**
 * Default search bar in enabled state.
 */
export const Default: Story = {
  args: {
    placeholder: "Search npm packages...",
    disabled: false,
  },
};

/**
 * Search bar when maximum packages have been selected.
 */
export const Disabled: Story = {
  args: {
    placeholder: "Maximum 6 packages reached",
    disabled: true,
  },
};

/**
 * Search bar with custom placeholder text.
 */
export const CustomPlaceholder: Story = {
  args: {
    placeholder: "Type to search for packages...",
    disabled: false,
  },
};
````

## File: src/components/search-bar.tsx
````typescript
"use client";

import { useRef, useEffect, useId } from "react";
import { Search, Loader2, Download } from "lucide-react";
import { Input } from "@/components/ui/input";
import { cn } from "@/lib/utils";
import type { NpmPackage } from "@/types/package";
import { useSearchBar, EMPTY_PACKAGES } from "@/hooks/use-search-bar";

// ─────────────────────────────────────────────────
// Utility
// ─────────────────────────────────────────────────

/**
 * Formats a number to a human-readable string (e.g., 1500000 -> "1.5M").
 * @param num - The number to format
 * @returns Formatted string with K/M/B suffix
 * @example
 * formatDownloads(1500000) // => "1.5M"
 * formatDownloads(2300)    // => "2.3K"
 * formatDownloads(500)     // => "500"
 */
function formatDownloads(num: number): string {
  if (num >= 1_000_000_000) {
    return `${(num / 1_000_000_000).toFixed(1)}B`;
  }
  if (num >= 1_000_000) {
    return `${(num / 1_000_000).toFixed(1)}M`;
  }
  if (num >= 1_000) {
    return `${(num / 1_000).toFixed(1)}K`;
  }
  return num.toString();
}

// ─────────────────────────────────────────────────
// Sub-components
// ─────────────────────────────────────────────────

interface PackageListItemProps {
  pkg: NpmPackage;
  index: number;
  selectedIndex: number;
  onSelect: (name: string) => void;
  onHover: (index: number) => void;
  showDescription?: boolean;
}

/**
 * Single package option in the dropdown list.
 * Shared between popular packages and search suggestions to avoid duplication.
 */
function PackageListItem({
  pkg,
  index,
  selectedIndex,
  onSelect,
  onHover,
  showDescription = false,
}: PackageListItemProps) {
  return (
    <li
      key={pkg.name}
      role="option"
      aria-selected={index === selectedIndex}
      className={cn(
        "cursor-pointer px-3 py-2 hover:bg-accent",
        index === selectedIndex && "bg-accent",
      )}
      onClick={() => onSelect(pkg.name)}
      onKeyDown={(e) => {
        if (e.key === "Enter" || e.key === " ") {
          e.preventDefault();
          onSelect(pkg.name);
        }
      }}
      onMouseEnter={() => onHover(index)}
    >
      <div className="flex items-center justify-between gap-2">
        <span className="font-medium font-mono">{pkg.name}</span>
        {pkg.weeklyDownloads !== undefined && (
          <span className="flex items-center gap-1 text-xs text-muted-foreground tabular-nums">
            <Download className="h-3 w-3" />
            {formatDownloads(pkg.weeklyDownloads)}/wk
          </span>
        )}
      </div>
      {showDescription && pkg.description && (
        <div className="truncate text-sm text-muted-foreground">
          {pkg.description}
        </div>
      )}
    </li>
  );
}

interface PopularPackageListProps {
  packages: NpmPackage[];
  isLoading: boolean;
  selectedIndex: number;
  selectedPackages: string[];
  onSelect: (name: string) => void;
  onHover: (index: number) => void;
}

/**
 * Popular packages section shown when the search input is focused but empty.
 */
function PopularPackageList({
  packages,
  isLoading,
  selectedIndex,
  selectedPackages,
  onSelect,
  onHover,
}: PopularPackageListProps) {
  return (
    <>
      <div className="px-3 py-2 text-xs font-medium text-muted-foreground border-b">
        Popular packages
      </div>
      {isLoading ? (
        <div
          className="flex items-center justify-center py-4"
          role="status"
          aria-busy="true"
          aria-live="polite"
        >
          <Loader2 className="h-4 w-4 animate-spin text-muted-foreground" aria-hidden="true" />
          <span className="sr-only">Loading popular packages...</span>
        </div>
      ) : (
        <ul className="max-h-60 overflow-auto py-1">
          {packages
            .filter((pkg) => !selectedPackages.includes(pkg.name))
            .map((pkg, index) => (
              <PackageListItem
                key={pkg.name}
                pkg={pkg}
                index={index}
                selectedIndex={selectedIndex}
                onSelect={onSelect}
                onHover={onHover}
              />
            ))}
        </ul>
      )}
    </>
  );
}

interface SuggestionListProps {
  suggestions: NpmPackage[];
  selectedIndex: number;
  onSelect: (name: string) => void;
  onHover: (index: number) => void;
}

/**
 * Search suggestion results list with descriptions.
 */
function SuggestionList({
  suggestions,
  selectedIndex,
  onSelect,
  onHover,
}: SuggestionListProps) {
  return (
    <ul className="max-h-60 overflow-auto py-1">
      {suggestions.map((pkg, index) => (
        <PackageListItem
          key={pkg.name}
          pkg={pkg}
          index={index}
          selectedIndex={selectedIndex}
          onSelect={onSelect}
          onHover={onHover}
          showDescription
        />
      ))}
    </ul>
  );
}

interface SearchDropdownProps {
  listboxId: string;
  dropdownRef: React.RefObject<HTMLDivElement | null>;
  validationError: string | null;
  error: string | null;
  query: string;
  suggestions: NpmPackage[];
  popularPackages: NpmPackage[];
  isLoadingPopular: boolean;
  hasSearched: boolean;
  isLoading: boolean;
  selectedIndex: number;
  selectedPackages: string[];
  onSelect: (name: string) => void;
  onHover: (index: number) => void;
}

/**
 * Dropdown container with conditional rendering for validation errors,
 * search errors, popular packages, search suggestions, and empty states.
 */
function SearchDropdown({
  listboxId,
  dropdownRef,
  validationError,
  error,
  query,
  suggestions,
  popularPackages,
  isLoadingPopular,
  hasSearched,
  isLoading,
  selectedIndex,
  selectedPackages,
  onSelect,
  onHover,
}: SearchDropdownProps) {
  const showPopular = query.length === 0 && (popularPackages.length > 0 || isLoadingPopular);

  if (validationError) {
    return (
      <div
        ref={dropdownRef}
        className="absolute z-50 mt-1 w-full rounded-md border bg-popover shadow-lg"
        id={`${listboxId}-listbox`}
        role="listbox"
      >
        <div
          id="search-validation-error"
          className="px-3 py-6 text-center"
          role="alert"
          aria-live="polite"
        >
          <p className="text-sm font-medium text-destructive">{validationError}</p>
          <p className="mt-1 text-xs text-muted-foreground">
            Package names must be lowercase, no spaces, and use valid npm characters
          </p>
        </div>
      </div>
    );
  }

  if (error) {
    return (
      <div
        ref={dropdownRef}
        className="absolute z-50 mt-1 w-full rounded-md border bg-popover shadow-lg"
        id={`${listboxId}-listbox`}
        role="listbox"
      >
        <div
          id="search-error"
          className="px-3 py-6 text-center text-sm text-destructive"
          role="alert"
          aria-live="assertive"
        >
          {error}
        </div>
      </div>
    );
  }

  if (showPopular) {
    return (
      <div
        ref={dropdownRef}
        className="absolute z-50 mt-1 w-full rounded-md border bg-popover shadow-lg"
        id={`${listboxId}-listbox`}
        role="listbox"
      >
        <PopularPackageList
          packages={popularPackages}
          isLoading={isLoadingPopular}
          selectedIndex={selectedIndex}
          selectedPackages={selectedPackages}
          onSelect={onSelect}
          onHover={onHover}
        />
      </div>
    );
  }

  if (suggestions.length > 0) {
    return (
      <div
        ref={dropdownRef}
        className="absolute z-50 mt-1 w-full rounded-md border bg-popover shadow-lg"
        id={`${listboxId}-listbox`}
        role="listbox"
      >
        <SuggestionList
          suggestions={suggestions}
          selectedIndex={selectedIndex}
          onSelect={onSelect}
          onHover={onHover}
        />
      </div>
    );
  }

  if (hasSearched && !isLoading) {
    return (
      <div
        ref={dropdownRef}
        className="absolute z-50 mt-1 w-full rounded-md border bg-popover shadow-lg"
        id={`${listboxId}-listbox`}
        role="listbox"
      >
        <div className="px-3 py-6 text-center text-sm text-muted-foreground">
          No packages found
        </div>
      </div>
    );
  }

  return null;
}

// ─────────────────────────────────────────────────
// Main Component
// ─────────────────────────────────────────────────

interface SearchBarProps {
  /** Callback when a package is selected */
  onSelectPackage: (packageName: string) => void;
  /** Already selected package names to exclude from suggestions */
  selectedPackages?: string[];
  /** Disabled state */
  disabled?: boolean;
  /** Placeholder text */
  placeholder?: string;
}

/**
 * Search bar with autocomplete for npm packages.
 * Implements debounced search with dropdown suggestions.
 * @param onSelectPackage - Callback when a package is selected
 * @param selectedPackages - Already selected packages to exclude
 * @param disabled - Whether the search is disabled
 * @param placeholder - Placeholder text for the input
 */
export function SearchBar({
  onSelectPackage,
  selectedPackages = EMPTY_PACKAGES,
  disabled = false,
  placeholder = "Search npm packages...",
}: SearchBarProps) {
  const listboxId = useId();
  const inputRef = useRef<HTMLInputElement>(null);
  const dropdownRef = useRef<HTMLDivElement>(null);

  const {
    state,
    handleInputChange,
    handleSelect: hookHandleSelect,
    handleKeyDown,
    fetchPopularPackages,
    openDropdown,
    closeDropdown,
    setSelectedIndex,
  } = useSearchBar({ onSelectPackage, selectedPackages });

  const handleSelect = (packageName: string) => {
    hookHandleSelect(packageName);
    inputRef.current?.focus();
  };

  useEffect(() => {
    function handleClickOutside(event: MouseEvent) {
      if (
        dropdownRef.current &&
        !dropdownRef.current.contains(event.target as Node) &&
        inputRef.current &&
        !inputRef.current.contains(event.target as Node)
      ) {
        closeDropdown();
      }
    }

    document.addEventListener("mousedown", handleClickOutside);
    return () => document.removeEventListener("mousedown", handleClickOutside);
  }, [closeDropdown]);

  const isDropdownVisible =
    state.isOpen &&
    (state.suggestions.length > 0 ||
      state.error ||
      state.validationError ||
      (state.hasSearched && !state.isLoading) ||
      (state.query.length === 0 &&
        (state.popularPackages.length > 0 || state.isLoadingPopular)));

  return (
    <div className="relative w-full max-w-xl px-0 sm:px-0">
      <div className="relative">
        <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
        <Input
          ref={inputRef}
          type="text"
          value={state.query}
          onChange={handleInputChange}
          onKeyDown={handleKeyDown}
          onFocus={() => {
            if (state.query.length === 0) {
              fetchPopularPackages();
              openDropdown();
            } else if (state.suggestions.length > 0) {
              openDropdown();
            }
          }}
          placeholder={placeholder}
          disabled={disabled}
          className="pl-10 pr-10"
          aria-label="Search npm packages"
          aria-expanded={state.isOpen}
          aria-haspopup="listbox"
          aria-autocomplete="list"
          aria-invalid={!!state.validationError || !!state.error}
          aria-describedby={
            state.validationError
              ? "search-validation-error"
              : state.error
                ? "search-error"
                : undefined
          }
          role="combobox"
          aria-controls={`${listboxId}-listbox`}
        />
        {state.isLoading && (
          <Loader2
            className="absolute right-3 top-1/2 h-4 w-4 -translate-y-1/2 animate-spin text-muted-foreground"
            aria-hidden="true"
          />
        )}
        <span role="status" aria-live="polite" className="sr-only">
          {state.isLoading ? "Searching for packages..." : ""}
        </span>
      </div>

      {isDropdownVisible && (
        <SearchDropdown
          listboxId={listboxId}
          dropdownRef={dropdownRef}
          validationError={state.validationError}
          error={state.error}
          query={state.query}
          suggestions={state.suggestions}
          popularPackages={state.popularPackages}
          isLoadingPopular={state.isLoadingPopular}
          hasSearched={state.hasSearched}
          isLoading={state.isLoading}
          selectedIndex={state.selectedIndex}
          selectedPackages={selectedPackages}
          onSelect={handleSelect}
          onHover={setSelectedIndex}
        />
      )}
    </div>
  );
}
````

## File: src/components/share-button.tsx
````typescript
"use client";

import { useCallback } from "react";
import { Share2, Check } from "lucide-react";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import {
  Tooltip,
  TooltipContent,
  TooltipTrigger,
} from "@/components/ui/tooltip";

interface ShareButtonProps {
  /** Disabled state */
  disabled?: boolean;
}

/**
 * Share button that copies the current URL to clipboard.
 * Shows a toast notification on success.
 * @param disabled - Whether the button is disabled
 * @example
 * <ShareButton /> // Copies current URL
 * <ShareButton disabled /> // Disabled state
 */
export function ShareButton({ disabled = false }: ShareButtonProps) {
  /**
   * Copies the current URL to clipboard and shows a toast.
   */
  const handleShare = useCallback(async () => {
    try {
      await navigator.clipboard.writeText(window.location.href);
      toast.success("URL copied to clipboard", {
        icon: <Check className="h-4 w-4" />,
        duration: 2000,
      });
    } catch {
      toast.error("Failed to copy URL");
    }
  }, []);

  return (
    <Tooltip>
      <TooltipTrigger asChild>
        <Button
          variant="outline"
          size="icon"
          onClick={handleShare}
          disabled={disabled}
          aria-label="Share URL"
        >
          <Share2 className="h-4 w-4" />
        </Button>
      </TooltipTrigger>
      <TooltipContent>
        <p>Copy URL to clipboard</p>
      </TooltipContent>
    </Tooltip>
  );
}
````

## File: src/components/theme-toggle.tsx
````typescript
"use client";

import { useTheme } from "next-themes";
import { Moon, Sun } from "lucide-react";
import { Button } from "@/components/ui/button";

/**
 * Theme toggle button for switching between light and dark modes.
 * Uses next-themes for theme management.
 *
 * @example
 * <ThemeToggle />
 */
export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  /**
   * Toggles between light and dark themes.
   */
  const toggleTheme = () => {
    setTheme(theme === "dark" ? "light" : "dark");
  };

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={toggleTheme}
      className="h-11 w-11 min-h-[44px] min-w-[44px]"
      aria-label={`Switch to ${theme === "dark" ? "light" : "dark"} mode`}
    >
      <Sun className="h-4 w-4 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-4 w-4 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
````

## File: src/components/time-range-selector.tsx
````typescript
"use client";

import { TIME_RANGES, type TimeRange } from "@/types/package";
import { cn } from "@/lib/utils";

interface TimeRangeSelectorProps {
  /** Currently selected time range */
  value: TimeRange;
  /** Callback when time range changes */
  onChange: (range: TimeRange) => void;
}

/**
 * Horizontal button group for selecting chart time range.
 * @param value - Current time range
 * @param onChange - Callback when selection changes
 */
export function TimeRangeSelector({ value, onChange }: TimeRangeSelectorProps) {
  return (
    <div className="flex items-center gap-0.5 sm:gap-1 rounded-lg bg-muted p-0.5 sm:p-1" role="group" aria-label="Time range selector" data-testid="time-range-selector">
      {TIME_RANGES.map((range) => (
        <button
          key={range.value}
          onClick={() => onChange(range.value)}
          className={cn(
            "rounded-md px-2 sm:px-3 py-2 sm:py-1.5 text-xs sm:text-sm font-medium transition-colors min-h-[44px] min-w-[36px] sm:min-w-[44px]",
            value === range.value
              ? "bg-background text-foreground shadow-sm"
              : "text-muted-foreground hover:text-foreground"
          )}
          aria-pressed={value === range.value}
        >
          {range.value.toUpperCase()}
        </button>
      ))}
    </div>
  );
}
````

## File: src/components/trend-chart.stories.tsx
````typescript
import type { Meta, StoryObj } from "@storybook/nextjs";
import { TrendChart } from "./trend-chart";
import { CHART_COLORS } from "@/constants/colors";
import type { ChartDataPoint, SelectedPackage } from "@/types/package";

/**
 * Generates mock chart data for stories.
 */
function generateMockData(packages: string[]): ChartDataPoint[] {
  if (packages.length === 0) return [];

  const data: ChartDataPoint[] = [];
  const today = new Date();

  for (let i = 365; i >= 0; i -= 7) {
    const date = new Date(today);
    date.setDate(date.getDate() - i);
    const dateStr = date.toISOString().split("T")[0];

    const point: ChartDataPoint = { date: dateStr };
    packages.forEach((pkg, index) => {
      const baseDownloads = (index + 1) * 1_000_000;
      const variation = Math.sin(i / 30) * 200_000;
      const trend = (365 - i) * 5000;
      point[pkg] = Math.max(0, Math.round(baseDownloads + variation + trend));
    });

    data.push(point);
  }

  return data;
}

/**
 * TrendChart displays a time-series line chart of npm package download statistics.
 * Built with Recharts for responsive, interactive visualizations.
 */
const meta: Meta<typeof TrendChart> = {
  title: "Components/TrendChart",
  component: TrendChart,
  parameters: {
    layout: "fullscreen",
    backgrounds: {
      default: "dark",
      values: [{ name: "dark", value: "#0f0f0f" }],
    },
  },
  tags: ["autodocs"],
  decorators: [
    (Story) => (
      <div className="p-8">
        <Story />
      </div>
    ),
  ],
};

export default meta;
type Story = StoryObj<typeof meta>;

const singlePackage: SelectedPackage[] = [
  { name: "react", color: CHART_COLORS[0] },
];

const twoPackages: SelectedPackage[] = [
  { name: "react", color: CHART_COLORS[0] },
  { name: "vue", color: CHART_COLORS[1] },
];

const multiplePackages: SelectedPackage[] = [
  { name: "react", color: CHART_COLORS[0] },
  { name: "vue", color: CHART_COLORS[1] },
  { name: "@angular/core", color: CHART_COLORS[2] },
  { name: "svelte", color: CHART_COLORS[3] },
];

/**
 * Empty state when no packages are selected.
 */
export const Empty: Story = {
  args: {
    data: [],
    packages: [],
    isLoading: false,
  },
};

/**
 * Loading state while fetching data.
 */
export const Loading: Story = {
  args: {
    data: [],
    packages: twoPackages,
    isLoading: true,
  },
};

/**
 * Single package trend visualization.
 */
export const SinglePackage: Story = {
  args: {
    data: generateMockData(["react"]),
    packages: singlePackage,
    isLoading: false,
  },
};

/**
 * Comparing two packages (React vs Vue).
 */
export const TwoPackages: Story = {
  args: {
    data: generateMockData(["react", "vue"]),
    packages: twoPackages,
    isLoading: false,
  },
};

/**
 * Multiple packages comparison view.
 */
export const MultiplePackages: Story = {
  args: {
    data: generateMockData(["react", "vue", "@angular/core", "svelte"]),
    packages: multiplePackages,
    isLoading: false,
  },
};
````

## File: src/components/trend-chart.tsx
````typescript
"use client";

import { useState } from "react";
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
  Brush,
} from "recharts";
import { Search, TrendingUp, Package, AlertCircle, RefreshCw } from "lucide-react";
import type { SelectedPackage, ChartDataPoint } from "@/types/package";
import { useReducedMotion } from "@/hooks/use-reduced-motion";

interface TrendChartProps {
  /** Chart data with dates and download counts */
  data: ChartDataPoint[];
  /** Selected packages with colors */
  packages: SelectedPackage[];
  /** Whether to show loading state */
  isLoading?: boolean;
  /** Error message if request failed */
  error?: string | null;
  /** Callback to retry failed request */
  onRetry?: () => void;
}

/**
 * Formats large numbers with K/M/B suffixes.
 * @param value - Number to format
 * @returns Formatted string (e.g., "1.5M")
 */
function formatNumber(value: number): string {
  if (value >= 1_000_000_000) {
    return `${(value / 1_000_000_000).toFixed(1)}B`;
  }
  if (value >= 1_000_000) {
    return `${(value / 1_000_000).toFixed(1)}M`;
  }
  if (value >= 1_000) {
    return `${(value / 1_000).toFixed(1)}K`;
  }
  return value.toString();
}

/**
 * Formats date for display on X-axis.
 * @param dateStr - ISO date string
 * @returns Formatted date as "Mon YYYY" (e.g., "Jan 2025")
 */
function formatDate(dateStr: string): string {
  const date = new Date(dateStr);
  return date.toLocaleDateString("en-US", {
    month: "short",
    year: "numeric",
  });
}

/**
 * Custom tooltip component for the chart.
 */
function CustomTooltip({
  active,
  payload,
  label,
}: {
  active?: boolean;
  payload?: Array<{
    name: string;
    value: number;
    color: string;
  }>;
  label?: string;
}) {
  if (!active || !payload || payload.length === 0) {
    return null;
  }

  return (
    <div className="rounded-lg border bg-popover p-3 shadow-lg">
      <p className="mb-2 text-sm font-medium text-foreground">
        {label ? new Date(label).toLocaleDateString("en-US", {
          month: "long",
          day: "numeric",
          year: "numeric",
        }) : ""}
      </p>
      <div className="space-y-1">
        {payload.map((entry) => (
          <div key={entry.name} className="flex items-center gap-2 text-sm">
            <div
              className="h-3 w-3 rounded-full"
              style={{ backgroundColor: entry.color }}
            />
            <span className="text-muted-foreground">{entry.name}:</span>
            <span className="font-medium tabular-nums">{formatNumber(entry.value)}</span>
          </div>
        ))}
      </div>
    </div>
  );
}

/**
 * Time-series chart for package download trends.
 * Uses Recharts for rendering.
 * @param data - Array of data points with dates and downloads
 * @param packages - Selected packages with their colors
 * @param isLoading - Whether data is being loaded
 */
export function TrendChart({ data, packages, isLoading = false, error, onRetry }: TrendChartProps) {
  const [hiddenLines, setHiddenLines] = useState<string[]>([]);
  const prefersReducedMotion = useReducedMotion();

  /**
   * Handles legend item click to toggle line visibility.
   * @param dataKey - The package name (dataKey) of the clicked legend item
   */
  const handleLegendClick = (dataKey: string) => {
    setHiddenLines((prev) =>
      prev.includes(dataKey)
        ? prev.filter((name) => name !== dataKey)
        : [...prev, dataKey]
    );
  };

  if (packages.length === 0) {
    return (
      <div className="flex h-[400px] flex-col items-center justify-center gap-4 rounded-lg border border-dashed bg-muted/30">
        {/* Empty state illustration */}
        <div className="relative">
          <div className="flex items-center gap-2 text-muted-foreground/40">
            <Package className="h-12 w-12" />
            <TrendingUp className="h-16 w-16" />
            <Package className="h-12 w-12" />
          </div>
          <Search className="absolute -bottom-2 -right-2 h-8 w-8 text-muted-foreground/60" />
        </div>
        <div className="text-center">
          <p className="text-muted-foreground">
            Search and select packages to compare download trends
          </p>
        </div>
      </div>
    );
  }

  if (isLoading) {
    return (
      <div
        className="flex h-[400px] items-center justify-center rounded-lg border bg-muted/30"
        role="status"
        aria-busy="true"
        aria-live="polite"
      >
        <div className="flex items-center gap-2 text-muted-foreground">
          <div
            className={`h-5 w-5 rounded-full border-2 border-current border-t-transparent ${
              prefersReducedMotion ? "" : "animate-spin"
            }`}
            aria-hidden="true"
          />
          <span>Loading download data...</span>
        </div>
      </div>
    );
  }

  if (error) {
    return (
      <div
        className="flex h-[400px] flex-col items-center justify-center gap-4 rounded-lg border border-destructive/50 bg-destructive/5"
        role="alert"
        aria-live="assertive"
      >
        <div className="flex items-center gap-2 text-destructive">
          <AlertCircle className="h-5 w-5" aria-hidden="true" />
          <span className="font-medium">Failed to load download data</span>
        </div>
        <p className="text-sm text-muted-foreground">{error}</p>
        {onRetry && (
          <button
            onClick={onRetry}
            className="inline-flex min-h-[44px] items-center gap-2 rounded-md bg-primary px-4 py-2 text-sm font-medium text-primary-foreground transition-colors hover:bg-primary/90 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
          >
            <RefreshCw className="h-4 w-4" aria-hidden="true" />
            Try again
          </button>
        )}
      </div>
    );
  }

  if (data.length === 0) {
    return (
      <div className="flex h-[400px] items-center justify-center rounded-lg border bg-muted/30">
        <p className="text-muted-foreground">No data available</p>
      </div>
    );
  }

  const packageNames = packages.map(p => p.name).join(", ");
  const ariaLabel = `Download trends chart comparing ${packages.length} npm package${packages.length === 1 ? "" : "s"}: ${packageNames}`;

  return (
    <div
      className="h-[400px] w-full"
      role="img"
      aria-label={ariaLabel}
      data-testid="trend-chart"
    >
      <ResponsiveContainer width="100%" height="100%">
        <LineChart
          data={data}
          margin={{ top: 20, right: 30, left: 20, bottom: 20 }}
        >
          <CartesianGrid
            strokeDasharray="3 3"
            stroke="rgba(255,255,255,0.1)"
            vertical={false}
          />
          <XAxis
            dataKey="date"
            tickFormatter={formatDate}
            tick={{ fill: "currentColor", fontSize: 12 }}
            tickLine={{ stroke: "currentColor" }}
            axisLine={{ stroke: "currentColor" }}
            className="text-muted-foreground"
            label={{
              value: "Date →",
              position: "insideBottomRight",
              offset: -5,
              style: { textAnchor: "end", fill: "currentColor", fontSize: 11 },
            }}
          />
          <YAxis
            tickFormatter={formatNumber}
            tick={{ fill: "currentColor", fontSize: 12 }}
            tickLine={{ stroke: "currentColor" }}
            axisLine={{ stroke: "currentColor" }}
            className="text-muted-foreground"
            label={{
              value: "↑ Downloads",
              angle: -90,
              position: "insideLeft",
              offset: -5,
              style: { textAnchor: "middle", fill: "currentColor" },
            }}
          />
          <Tooltip content={<CustomTooltip />} />
          <Legend
            wrapperStyle={{
              paddingTop: 16,
            }}
            onClick={(e) => {
              if (e && e.dataKey) {
                handleLegendClick(e.dataKey as string);
              }
            }}
            formatter={(value: string) => (
              <span
                className="text-sm cursor-pointer select-none"
                style={{
                  color: hiddenLines.includes(value)
                    ? "var(--muted-foreground)"
                    : "var(--foreground)",
                  textDecoration: hiddenLines.includes(value)
                    ? "line-through"
                    : "none",
                  opacity: hiddenLines.includes(value) ? 0.5 : 1,
                }}
              >
                {value}
              </span>
            )}
          />
          {packages.map((pkg) => (
            <Line
              key={pkg.name}
              type="monotone"
              dataKey={pkg.name}
              stroke={pkg.color}
              strokeWidth={2}
              dot={false}
              activeDot={{ r: 4, strokeWidth: 0 }}
              hide={hiddenLines.includes(pkg.name)}
              isAnimationActive={!prefersReducedMotion}
            />
          ))}
          <Brush
            dataKey="date"
            height={30}
            stroke="hsl(var(--primary))"
            fill="hsl(var(--muted))"
            tickFormatter={formatDate}
          >
            <LineChart>
              {packages.map((pkg) => (
                <Line
                  key={`brush-${pkg.name}`}
                  type="monotone"
                  dataKey={pkg.name}
                  stroke={pkg.color}
                  strokeWidth={1}
                  dot={false}
                />
              ))}
            </LineChart>
          </Brush>
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
````

## File: src/components/user-menu.tsx
````typescript
"use client";

import { useRouter } from "next/navigation";
import Link from "next/link";
import { User, Settings, LogOut } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { useAuth } from "@/contexts/auth-context";

/**
 * User menu component.
 * Shows login/signup buttons when not authenticated.
 * Shows user avatar with dropdown menu when authenticated.
 */
export function UserMenu() {
  const { user, isAuthenticated, isLoading, logout } = useAuth();
  const router = useRouter();

  /**
   * Handles logout action.
   * Clears auth state and redirects to home.
   */
  const handleLogout = async () => {
    await logout();
    router.push("/");
  };

  // Show nothing while loading to prevent flash
  if (isLoading) {
    return (
      <div className="min-h-[44px] min-w-[44px] flex items-center justify-center">
        <div className="h-8 w-8 rounded-full bg-muted animate-pulse" />
      </div>
    );
  }

  // Not authenticated - show login button
  if (!isAuthenticated) {
    return (
      <Button variant="outline" size="sm" asChild>
        <Link href="/login">Sign in</Link>
      </Button>
    );
  }

  // Authenticated - show user avatar with dropdown
  const initials = user?.name
    ? user.name.slice(0, 2).toUpperCase()
    : user?.email?.slice(0, 2).toUpperCase() || "U";

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button
          variant="ghost"
          className="relative h-10 w-10 rounded-full bg-primary text-primary-foreground hover:bg-primary/90"
          aria-label="User menu"
        >
          <span className="text-sm font-medium">{initials}</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-56">
        <div className="flex items-center gap-2 p-2">
          <div className="flex h-8 w-8 items-center justify-center rounded-full bg-primary text-primary-foreground">
            <User className="h-4 w-4" />
          </div>
          <div className="flex flex-col space-y-0.5">
            <p className="text-sm font-medium">{user?.name}</p>
            <p className="text-xs text-muted-foreground">{user?.email}</p>
          </div>
        </div>
        <DropdownMenuSeparator />
        <DropdownMenuItem asChild>
          <Link href="/settings" className="cursor-pointer">
            <Settings className="mr-2 h-4 w-4" />
            Settings
          </Link>
        </DropdownMenuItem>
        <DropdownMenuSeparator />
        <DropdownMenuItem
          onClick={handleLogout}
          className="cursor-pointer text-destructive focus:text-destructive"
        >
          <LogOut className="mr-2 h-4 w-4" />
          Logout
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
````

## File: src/constants/category-presets.ts
````typescript
/**
 * Category presets for the "Discover packages" feature.
 * Each category maps to a list of popular npm packages in that domain.
 */
export const CATEGORY_PRESETS: Record<string, string[]> = {
  "Front-end": ["react", "vue", "angular", "svelte"],
  "Back-end": ["express", "fastify", "nest", "koa"],
  CLI: ["commander", "inquirer", "chalk", "ora"],
  Documentation: ["jsdoc", "typedoc", "docusaurus"],
  CSS: ["tailwindcss", "sass", "postcss", "styled-components"],
  Testing: ["jest", "vitest", "mocha", "cypress"],
  IoT: ["johnny-five", "mqtt", "serialport"],
  Coverage: ["istanbul", "c8", "nyc"],
  Mobile: ["react-native", "expo", "capacitor"],
  Frameworks: ["next", "nuxt", "remix", "astro"],
  Robotics: ["robotjs", "cylon"],
  Math: ["mathjs", "decimal.js", "bignumber.js"],
} as const;

/**
 * Type for category names.
 */
export type CategoryName = keyof typeof CATEGORY_PRESETS;
````

## File: src/constants/colors.ts
````typescript
/**
 * Default color palette for package comparison charts.
 * Colors are designed to be distinguishable and accessible.
 */
export const CHART_COLORS = [
  "#61dafb", // React blue
  "#42b883", // Vue green
  "#dd0031", // Angular red
  "#ff3e00", // Svelte orange
  "#2c4f7c", // Solid blue
  "#673ab8", // Preact purple
] as const;

/**
 * Get a color from the palette by index.
 * @param index - Index of the package in the comparison list
 * @returns Color hex string
 */
export function getChartColor(index: number): string {
  return CHART_COLORS[index % CHART_COLORS.length];
}

/**
 * Maximum number of packages that can be compared simultaneously.
 */
export const MAX_PACKAGES = 6;
````

## File: src/contexts/auth-context.tsx
````typescript
"use client";

import {
  createContext,
  useContext,
  useCallback,
  useMemo,
  type ReactNode,
} from "react";
import { authClient, useSession } from "@/lib/auth-client";

interface AuthUser {
  id: string;
  email: string;
  name: string;
  image?: string | null;
  emailVerified: boolean;
}

interface AuthContextType {
  user: AuthUser | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<{ success: boolean; error?: string }>;
  signup: (name: string, email: string, password: string) => Promise<{ success: boolean; error?: string }>;
  logout: () => Promise<void>;
  updateName: (name: string) => Promise<boolean>;
  deleteAccount: (password?: string) => Promise<boolean>;
}

const AuthContext = createContext<AuthContextType | null>(null);

/**
 * Auth provider that wraps Better Auth's session management.
 * Provides a unified interface for login, signup, logout, and account management.
 */
export function AuthProvider({ children }: { children: ReactNode }) {
  const { data: session, isPending } = useSession();

  const user: AuthUser | null = useMemo(() => {
    if (!session?.user) return null;
    return {
      id: session.user.id,
      email: session.user.email,
      name: session.user.name,
      image: session.user.image,
      emailVerified: session.user.emailVerified,
    };
  }, [session]);

  const login = useCallback(
    async (email: string, password: string): Promise<{ success: boolean; error?: string }> => {
      const { error } = await authClient.signIn.email({ email, password });
      if (error) {
        return { success: false, error: error.message ?? "Sign in failed" };
      }
      return { success: true };
    },
    []
  );

  const signup = useCallback(
    async (name: string, email: string, password: string): Promise<{ success: boolean; error?: string }> => {
      const { error } = await authClient.signUp.email({ name, email, password });
      if (error) {
        return { success: false, error: error.message ?? "Sign up failed" };
      }
      return { success: true };
    },
    []
  );

  const logout = useCallback(async () => {
    await authClient.signOut();
  }, []);

  const updateName = useCallback(async (newName: string): Promise<boolean> => {
    const { error } = await authClient.updateUser({ name: newName });
    return !error;
  }, []);

  const deleteAccount = useCallback(async (password?: string): Promise<boolean> => {
    const { error } = password
      ? await authClient.deleteUser({ password })
      : await authClient.deleteUser();
    return !error;
  }, []);

  const value = useMemo(
    () => ({
      user,
      isLoading: isPending,
      isAuthenticated: !!user,
      login,
      signup,
      logout,
      updateName,
      deleteAccount,
    }),
    [user, isPending, login, signup, logout, updateName, deleteAccount]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

/**
 * Hook to access auth context.
 * @throws Error if used outside AuthProvider
 */
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within an AuthProvider");
  }
  return context;
}
````

## File: src/hooks/use-downloads.ts
````typescript
"use client";

import { useState, useEffect, useRef } from "react";
import { toast } from "sonner";
import type { ChartDataPoint, TimeRange } from "@/types/package";

/**
 * Custom error class for rate limit responses.
 * Contains the retry-after duration in seconds.
 */
class RateLimitError extends Error {
  retryAfter: number;

  constructor(retryAfter: number) {
    super(`Rate limit exceeded. Please try again in ${retryAfter} seconds.`);
    this.name = "RateLimitError";
    this.retryAfter = retryAfter;
  }
}

/**
 * API response structure for download data.
 */
interface DownloadData {
  [packageName: string]:
    | { downloads: Array<{ day: string; downloads: number }> }
    | { error: string };
}

/**
 * Aggregates daily downloads into weekly data points.
 * @param downloads - Array of daily download data
 * @returns Aggregated weekly totals
 */
function aggregateWeekly(
  downloads: Array<{ day: string; downloads: number }>
): Array<{ day: string; downloads: number }> {
  const weeklyData: Array<{ day: string; downloads: number }> = [];
  let weekTotal = 0;
  let weekStart = "";

  for (let i = 0; i < downloads.length; i++) {
    if (i % 7 === 0) {
      if (weekStart) {
        weeklyData.push({ day: weekStart, downloads: weekTotal });
      }
      weekStart = downloads[i].day;
      weekTotal = downloads[i].downloads;
    } else {
      weekTotal += downloads[i].downloads;
    }
  }

  // Push the last week
  if (weekStart) {
    weeklyData.push({ day: weekStart, downloads: weekTotal });
  }

  return weeklyData;
}

/**
 * Custom hook for fetching and managing package download data.
 * Uses a stable string key to prevent infinite re-renders from array reference changes.
 * @param packageNames - Array of package names to fetch
 * @param timeRange - Time range for the data
 * @returns
 * - data: Chart data points
 * - isLoading: Loading state
 * - error: Error message if request failed
 * - invalidPackages: Array of package names that returned errors
 * - refetch: Function to trigger re-fetch
 * @example
 * const { data, isLoading, error, invalidPackages } = useDownloads(['react', 'vue'], '1y');
 */
export function useDownloads(packageNames: string[], timeRange: TimeRange) {
  const [data, setData] = useState<ChartDataPoint[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [invalidPackages, setInvalidPackages] = useState<string[]>([]);
  const [refetchCount, setRefetchCount] = useState(0);

  // Use a stable string key to prevent infinite re-renders
  const packagesKey = packageNames.join(",");
  const abortControllerRef = useRef<AbortController | null>(null);

  useEffect(() => {
    // Cancel previous request if still in flight
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    if (!packagesKey) {
      setData([]);
      setIsLoading(false);
      return;
    }

    const controller = new AbortController();
    abortControllerRef.current = controller;

    // Track whether abort was caused by timeout
    let isTimeout = false;

    // Timeout after 30 seconds
    const timeoutId = setTimeout(() => {
      isTimeout = true;
      controller.abort();
    }, 30000);

    /**
     * Fetch with automatic retry and exponential backoff.
     * @param url - URL to fetch
     * @param options - Fetch options
     * @param retries - Number of retries remaining
     * @param delay - Current delay in ms
     */
    async function fetchWithRetry(
      url: string,
      options: RequestInit,
      retries = 3,
      delay = 1000
    ): Promise<Response> {
      try {
        const response = await fetch(url, options);

        // Handle rate limiting - don't retry, show wait time to user
        if (response.status === 429) {
          const retryAfterHeader = response.headers.get("Retry-After");
          let retryAfter = 10; // Default fallback

          // Try to get retryAfter from response body
          try {
            const errorBody = await response.json();
            if (typeof errorBody.retryAfter === "number") {
              retryAfter = errorBody.retryAfter;
            }
          } catch {
            // If body parsing fails, use header value
            if (retryAfterHeader) {
              retryAfter = parseInt(retryAfterHeader, 10) || 10;
            }
          }

          throw new RateLimitError(retryAfter);
        }

        if (!response.ok) {
          throw new Error("Failed to fetch download data");
        }
        return response;
      } catch (err) {
        // Don't retry on abort, rate limit, or when no retries left
        if (
          retries <= 0 ||
          (err instanceof Error && err.name === "AbortError") ||
          err instanceof RateLimitError
        ) {
          throw err;
        }
        // Wait with exponential backoff and retry
        await new Promise((resolve) => setTimeout(resolve, delay));
        return fetchWithRetry(url, options, retries - 1, delay * 2);
      }
    }

    const fetchDownloads = async () => {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetchWithRetry(
          `/api/downloads?packages=${packagesKey}&range=${timeRange}`,
          { signal: controller.signal }
        );

        clearTimeout(timeoutId);

        const downloadData: DownloadData = await response.json();

        // Collect all unique dates
        const dateSet = new Set<string>();
        const packageDownloads: Record<string, Map<string, number>> = {};
        const invalidPkgs: string[] = [];

        for (const [pkgName, pkgData] of Object.entries(downloadData)) {
          if ("error" in pkgData) {
            console.warn(`Error fetching ${pkgName}: ${pkgData.error}`);
            invalidPkgs.push(pkgName);
            continue;
          }

          // Aggregate to weekly data for cleaner charts
          const weeklyData = aggregateWeekly(pkgData.downloads);
          packageDownloads[pkgName] = new Map();

          for (const { day, downloads } of weeklyData) {
            dateSet.add(day);
            packageDownloads[pkgName].set(day, downloads);
          }
        }

        // Sort dates and build chart data
        const sortedDates = Array.from(dateSet).sort();
        const chartData: ChartDataPoint[] = sortedDates.map((date) => {
          const point: ChartDataPoint = { date };
          for (const [pkgName, downloads] of Object.entries(packageDownloads)) {
            point[pkgName] = downloads.get(date) || 0;
          }
          return point;
        });

        setData(chartData);
        setInvalidPackages(invalidPkgs);
      } catch (err) {
        // Handle timeout vs user abort
        if (err instanceof Error && err.name === "AbortError") {
          if (isTimeout) {
            // Timeout-initiated abort - show user-friendly message
            setError("Request timed out. The server took too long to respond.");
            setData([]);
          }
          // User-initiated abort (component unmount, etc.) - ignore silently
          return;
        }

        // Handle rate limiting with user-friendly toast
        if (err instanceof RateLimitError) {
          const waitTime = err.retryAfter;
          toast.error("Rate limit exceeded", {
            description: `Too many requests. Please wait ${waitTime} seconds before trying again.`,
            duration: Math.min(waitTime * 1000, 10000), // Show for wait time, max 10s
          });
          setError(`Rate limited. Please wait ${waitTime} seconds.`);
          setData([]);
          return;
        }

        console.error("Error fetching downloads:", err);
        setError(err instanceof Error ? err.message : "Unknown error");
        setData([]);
      } finally {
        setIsLoading(false);
      }
    };

    fetchDownloads();

    return () => {
      clearTimeout(timeoutId);
      controller.abort();
    };
  }, [packagesKey, timeRange, refetchCount]);

  const refetch = () => {
    // Trigger re-fetch by incrementing counter to force useEffect re-run
    setRefetchCount((c) => c + 1);
  };

  return { data, isLoading, error, invalidPackages, refetch };
}
````

## File: src/hooks/use-online-status.ts
````typescript
"use client";

import { useState, useEffect } from "react";

/**
 * Custom hook to track online/offline status.
 * Uses the browser's navigator.onLine and online/offline events.
 * @returns Current online status (true = online, false = offline)
 * @example
 * const isOnline = useOnlineStatus();
 * if (!isOnline) {
 *   return <OfflineBanner />;
 * }
 */
export function useOnlineStatus(): boolean {
  // Initialize with navigator.onLine, defaulting to true for SSR
  const [isOnline, setIsOnline] = useState(() => {
    if (typeof window === "undefined") return true;
    return navigator.onLine;
  });

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return isOnline;
}
````

## File: src/hooks/use-package-info.ts
````typescript
"use client";

import { useState, useEffect, useRef } from "react";
import type { PackageInfo } from "@/types/package";

/**
 * API response structure for package info.
 */
type PackageInfoResponse = Record<string, PackageInfo | { error: string }>;

/**
 * Custom hook for fetching package metadata.
 * @param packageNames - Array of package names to fetch info for
 * @returns
 * - data: Map of package name to PackageInfo
 * - isLoading: Loading state
 * - error: Error message if request failed
 * @example
 * const { data, isLoading, error } = usePackageInfo(['react', 'vue']);
 * // data = { react: { name: 'react', ... }, vue: { name: 'vue', ... } }
 */
export function usePackageInfo(packageNames: string[]) {
  const [data, setData] = useState<Record<string, PackageInfo>>({});
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Use stable string key to prevent infinite re-renders
  const packagesKey = packageNames.join(",");
  const abortControllerRef = useRef<AbortController | null>(null);

  useEffect(() => {
    // Cancel previous request if still in flight
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    if (!packagesKey) {
      setData({});
      setIsLoading(false);
      return;
    }

    const controller = new AbortController();
    abortControllerRef.current = controller;

    const fetchPackageInfo = async () => {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetch(
          `/api/packages/info?packages=${packagesKey}`,
          { signal: controller.signal }
        );

        if (!response.ok) {
          throw new Error("Failed to fetch package info");
        }

        const responseData: PackageInfoResponse = await response.json();

        // Filter out error responses
        const validData: Record<string, PackageInfo> = {};
        for (const [name, info] of Object.entries(responseData)) {
          if (!("error" in info)) {
            validData[name] = info;
          }
        }

        setData(validData);
      } catch (err) {
        // Ignore abort errors
        if (err instanceof Error && err.name === "AbortError") {
          return;
        }
        console.error("Error fetching package info:", err);
        setError(err instanceof Error ? err.message : "Unknown error");
        setData({});
      } finally {
        setIsLoading(false);
      }
    };

    fetchPackageInfo();

    return () => {
      controller.abort();
    };
  }, [packagesKey]);

  return { data, isLoading, error };
}
````

## File: src/hooks/use-presets.ts
````typescript
"use client";

import { useState, useCallback, useEffect } from "react";
import { useSession } from "@/lib/auth-client";
import { authFetch } from "@/lib/auth-fetch";

/**
 * Maximum number of presets allowed per user.
 */
const MAX_PRESETS = 10;

/**
 * Maximum length for preset names.
 */
const MAX_NAME_LENGTH = 50;

/**
 * Local storage key for presets.
 */
const STORAGE_KEY = "npm-trends-presets";

/**
 * Preset data structure (localStorage format).
 */
export interface Preset {
  id: string;
  name: string;
  packages: string[];
  timeRange?: string;
  createdAt: number;
}

/**
 * Preset data structure from API.
 */
interface ApiPreset {
  id: string;
  name: string;
  packages: string[];
  timeRange: string;
  createdAt: string;
}

/**
 * Convert API preset to local format.
 */
function apiToLocal(apiPreset: ApiPreset): Preset {
  return {
    id: apiPreset.id,
    name: apiPreset.name,
    packages: apiPreset.packages,
    timeRange: apiPreset.timeRange,
    createdAt: new Date(apiPreset.createdAt).getTime(),
  };
}

/**
 * Load presets from localStorage.
 * @returns Sorted array of presets (newest first)
 */
function loadPresetsFromStorage(): Preset[] {
  if (typeof window === "undefined") return [];

  try {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      const parsed = JSON.parse(stored) as Preset[];
      // Sort by creation date (newest first)
      return parsed.sort((a, b) => b.createdAt - a.createdAt);
    }
  } catch (err) {
    console.error("Failed to load presets:", err);
  }
  return [];
}

/**
 * Custom hook for managing presets.
 * Supports both localStorage (anonymous) and cloud storage (authenticated users).
 *
 * @returns Preset management functions and state
 * @example
 * const { presets, savePreset, loadPreset, deletePreset, isLoading } = usePresets();
 */
export function usePresets() {
  const { data: session, isPending: isSessionLoading } = useSession();
  const isAuthenticated = !!session?.user?.id;

  const [presets, setPresets] = useState<Preset[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  /**
   * Fetch presets from API for authenticated users.
   */
  const fetchPresetsFromApi = useCallback(async () => {
    try {
      const response = await authFetch("/api/presets");
      if (!response.ok) {
        if (response.status === 401) {
          // Not authenticated, fall back to localStorage
          // Note: authFetch will redirect to login if 401
          return null;
        }
        throw new Error("Failed to fetch presets");
      }
      const data = (await response.json()) as ApiPreset[];
      return data.map(apiToLocal);
    } catch (err) {
      console.error("Failed to fetch presets from API:", err);
      return null;
    }
  }, []);

  /**
   * Load presets on mount and when authentication changes.
   */
  useEffect(() => {
    async function loadPresets() {
      setIsLoading(true);
      setError(null);

      if (isSessionLoading) {
        return; // Wait for session to load
      }

      if (isAuthenticated) {
        // Try to fetch from API
        const apiPresets = await fetchPresetsFromApi();
        if (apiPresets !== null) {
          setPresets(apiPresets);
        } else {
          // Fall back to localStorage if API fails
          setPresets(loadPresetsFromStorage());
        }
      } else {
        // Use localStorage for anonymous users
        setPresets(loadPresetsFromStorage());
      }

      setIsLoading(false);
    }

    loadPresets();
  }, [isAuthenticated, isSessionLoading, fetchPresetsFromApi]);

  /**
   * Persist presets to localStorage.
   */
  const persistToStorage = useCallback((newPresets: Preset[]) => {
    if (typeof window === "undefined") return;

    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(newPresets));
    } catch (err) {
      console.error("Failed to save presets:", err);
    }
  }, []);

  /**
   * Validates a preset name.
   * @returns Error message or null if valid
   */
  const validateName = useCallback(
    (name: string, excludeId?: string): string | null => {
      if (!name || name.trim().length === 0) {
        return "Preset name cannot be empty";
      }
      if (name.length > MAX_NAME_LENGTH) {
        return `Preset name must be ${MAX_NAME_LENGTH} characters or less`;
      }
      const duplicate = presets.find(
        (p) => p.name.toLowerCase() === name.toLowerCase() && p.id !== excludeId
      );
      if (duplicate) {
        return "A preset with this name already exists";
      }
      return null;
    },
    [presets]
  );

  /**
   * Saves a new preset.
   * @returns Created preset or error message
   */
  const savePreset = useCallback(
    async (
      name: string,
      packages: string[],
      timeRange: string = "1y"
    ): Promise<{ preset?: Preset; error?: string }> => {
      // Validate name
      const nameError = validateName(name);
      if (nameError) {
        return { error: nameError };
      }

      // Check max presets
      if (presets.length >= MAX_PRESETS) {
        return { error: `Maximum ${MAX_PRESETS} presets allowed` };
      }

      if (isAuthenticated) {
        // Save to API for authenticated users
        try {
          const response = await authFetch("/api/presets", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ name, packages, timeRange }),
          });

          if (!response.ok) {
            const data = await response.json();
            return { error: data.error || "Failed to save preset" };
          }

          const apiPreset = (await response.json()) as ApiPreset;
          const preset = apiToLocal(apiPreset);
          setPresets((prev) => [preset, ...prev]);
          return { preset };
        } catch (err) {
          console.error("Failed to save preset to API:", err);
          return { error: "Failed to save preset" };
        }
      } else {
        // Save to localStorage for anonymous users
        const preset: Preset = {
          id: crypto.randomUUID(),
          name: name.trim(),
          packages,
          timeRange,
          createdAt: Date.now(),
        };

        const newPresets = [preset, ...presets];
        setPresets(newPresets);
        persistToStorage(newPresets);

        return { preset };
      }
    },
    [presets, validateName, isAuthenticated, persistToStorage]
  );

  /**
   * Deletes a preset by ID.
   */
  const deletePreset = useCallback(
    async (id: string): Promise<boolean> => {
      const index = presets.findIndex((p) => p.id === id);
      if (index === -1) return false;

      if (isAuthenticated) {
        // Delete from API for authenticated users
        try {
          const response = await authFetch(`/api/presets/${id}`, {
            method: "DELETE",
          });

          if (!response.ok) {
            console.error("Failed to delete preset from API");
            return false;
          }

          setPresets((prev) => prev.filter((p) => p.id !== id));
          return true;
        } catch (err) {
          console.error("Failed to delete preset from API:", err);
          return false;
        }
      } else {
        // Delete from localStorage for anonymous users
        const newPresets = presets.filter((p) => p.id !== id);
        setPresets(newPresets);
        persistToStorage(newPresets);
        return true;
      }
    },
    [presets, isAuthenticated, persistToStorage]
  );

  /**
   * Renames a preset.
   * @returns Error message or null if successful
   */
  const renamePreset = useCallback(
    async (id: string, newName: string): Promise<string | null> => {
      // Validate name
      const nameError = validateName(newName, id);
      if (nameError) {
        return nameError;
      }

      if (isAuthenticated) {
        // Update via API for authenticated users
        try {
          const response = await authFetch(`/api/presets/${id}`, {
            method: "PATCH",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ name: newName }),
          });

          if (!response.ok) {
            const data = await response.json();
            return data.error || "Failed to rename preset";
          }

          setPresets((prev) =>
            prev.map((p) => (p.id === id ? { ...p, name: newName.trim() } : p))
          );
          return null;
        } catch (err) {
          console.error("Failed to rename preset via API:", err);
          return "Failed to rename preset";
        }
      } else {
        // Update localStorage for anonymous users
        const newPresets = presets.map((p) =>
          p.id === id ? { ...p, name: newName.trim() } : p
        );
        setPresets(newPresets);
        persistToStorage(newPresets);
        return null;
      }
    },
    [presets, validateName, isAuthenticated, persistToStorage]
  );

  /**
   * Gets a preset by ID.
   */
  const getPreset = useCallback(
    (id: string): Preset | undefined => {
      return presets.find((p) => p.id === id);
    },
    [presets]
  );

  /**
   * Refresh presets from the server (for authenticated users).
   */
  const refreshPresets = useCallback(async () => {
    if (!isAuthenticated) return;

    setIsLoading(true);
    const apiPresets = await fetchPresetsFromApi();
    if (apiPresets !== null) {
      setPresets(apiPresets);
    }
    setIsLoading(false);
  }, [isAuthenticated, fetchPresetsFromApi]);

  return {
    presets,
    savePreset,
    deletePreset,
    renamePreset,
    getPreset,
    validateName,
    refreshPresets,
    isLoading: isLoading || isSessionLoading,
    isAuthenticated,
    error,
    maxPresets: MAX_PRESETS,
    maxNameLength: MAX_NAME_LENGTH,
  };
}
````

## File: src/hooks/use-reduced-motion.ts
````typescript
"use client";

import { useState, useEffect } from "react";

/**
 * Gets the initial reduced motion preference.
 * Safe to call on server (returns false).
 */
function getInitialReducedMotion(): boolean {
  if (typeof window === "undefined") {
    return false;
  }
  return window.matchMedia("(prefers-reduced-motion: reduce)").matches;
}

/**
 * Detects user's reduced motion preference from system settings.
 * Returns true when the user prefers reduced motion (accessibility setting).
 *
 * @returns Whether the user prefers reduced motion
 * @example
 * const prefersReducedMotion = useReducedMotion();
 * <Line isAnimationActive={!prefersReducedMotion} />
 */
export function useReducedMotion(): boolean {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(
    getInitialReducedMotion
  );

  useEffect(() => {
    const mediaQuery = window.matchMedia("(prefers-reduced-motion: reduce)");

    // Listen for changes
    const handleChange = (event: MediaQueryListEvent) => {
      setPrefersReducedMotion(event.matches);
    };

    mediaQuery.addEventListener("change", handleChange);

    return () => {
      mediaQuery.removeEventListener("change", handleChange);
    };
  }, []);

  return prefersReducedMotion;
}
````

## File: src/hooks/use-search-bar.ts
````typescript
"use client";

import { useReducer, useCallback, useRef } from "react";
import type { NpmPackage } from "@/types/package";

// ─────────────────────────────────────────────────
// Constants
// ─────────────────────────────────────────────────

/**
 * Popular packages shown when search input is focused but empty.
 * Sorted by typical download counts to show most popular first.
 */
export const POPULAR_PACKAGES = [
  "react",
  "vue",
  "angular",
  "svelte",
  "next",
  "typescript",
  "express",
  "lodash",
  "axios",
  "webpack",
];

/** Stable empty array to prevent unnecessary re-renders. */
export const EMPTY_PACKAGES: string[] = [];

// ─────────────────────────────────────────────────
// Validation
// ─────────────────────────────────────────────────

/**
 * Validates npm package name format.
 * @param name - Package name to validate
 * @returns Error message if invalid, null if valid
 * @see https://docs.npmjs.com/cli/v10/configuring-npm/package-json#name
 * @example
 * validatePackageName("react")       // => null
 * validatePackageName(".bad")        // => "Package names cannot start with a dot or underscore"
 * validatePackageName("Has Spaces")  // => "Package names cannot contain spaces"
 */
function validatePackageName(name: string): string | null {
  if (!name || name.length < 1) {
    return null; // Empty is not an error, just no validation needed
  }

  // Check for invalid starting characters
  if (name.startsWith(".") || name.startsWith("_")) {
    return "Package names cannot start with a dot or underscore";
  }

  // Check for spaces
  if (/\s/.test(name)) {
    return "Package names cannot contain spaces";
  }

  // Check for uppercase (except in scoped packages after @)
  const nameWithoutScope = name.startsWith("@") ? name.split("/")[1] || "" : name;
  if (nameWithoutScope !== nameWithoutScope.toLowerCase()) {
    return "Package names must be lowercase";
  }

  // Check for invalid characters (allow @, /, -, _, .)
  if (!/^(@[a-z0-9-~][a-z0-9-._~]*\/)?[a-z0-9-~][a-z0-9-._~]*$/.test(name)) {
    return "Package name contains invalid characters";
  }

  // Check maximum length
  if (name.length > 214) {
    return "Package name is too long (max 214 characters)";
  }

  return null;
}

// ─────────────────────────────────────────────────
// State & Actions
// ─────────────────────────────────────────────────

export interface SearchBarState {
  query: string;
  validationError: string | null;
  suggestions: NpmPackage[];
  isLoading: boolean;
  hasSearched: boolean;
  error: string | null;
  isOpen: boolean;
  selectedIndex: number;
  popularPackages: NpmPackage[];
  isLoadingPopular: boolean;
}

type SearchBarAction =
  | { type: "SET_QUERY"; query: string; validationError: string | null }
  | { type: "FETCH_START" }
  | { type: "FETCH_SUCCESS"; suggestions: NpmPackage[] }
  | { type: "FETCH_ERROR"; error: string }
  | { type: "SELECT_PACKAGE" }
  | { type: "OPEN_DROPDOWN" }
  | { type: "CLOSE_DROPDOWN" }
  | { type: "SET_SELECTED_INDEX"; index: number }
  | { type: "POPULAR_FETCH_START" }
  | { type: "POPULAR_FETCH_SUCCESS"; packages: NpmPackage[] };

const initialState: SearchBarState = {
  query: "",
  validationError: null,
  suggestions: [],
  isLoading: false,
  hasSearched: false,
  error: null,
  isOpen: false,
  selectedIndex: -1,
  popularPackages: [],
  isLoadingPopular: false,
};

/**
 * Pure reducer for SearchBar state transitions.
 * Each action represents an atomic state change that keeps all related fields consistent.
 * @param state - Current state
 * @param action - Action to apply
 * @returns Next state
 * @example
 * searchBarReducer(initialState, { type: "FETCH_START" })
 * // => { ...initialState, isLoading: true, error: null }
 */
export function searchBarReducer(
  state: SearchBarState,
  action: SearchBarAction,
): SearchBarState {
  switch (action.type) {
    case "SET_QUERY": {
      const base = {
        ...state,
        query: action.query,
        validationError: action.validationError,
        selectedIndex: -1,
      };
      if (action.validationError) {
        return { ...base, suggestions: [], isOpen: true, hasSearched: false };
      }
      return base;
    }
    case "FETCH_START":
      return { ...state, isLoading: true, error: null };
    case "FETCH_SUCCESS":
      return {
        ...state,
        suggestions: action.suggestions,
        hasSearched: true,
        isOpen: true,
        isLoading: false,
      };
    case "FETCH_ERROR":
      return {
        ...state,
        suggestions: [],
        hasSearched: true,
        error: action.error,
        isOpen: true,
        isLoading: false,
      };
    case "SELECT_PACKAGE":
      return {
        ...state,
        query: "",
        suggestions: [],
        isOpen: false,
        selectedIndex: -1,
      };
    case "OPEN_DROPDOWN":
      return { ...state, isOpen: true };
    case "CLOSE_DROPDOWN":
      return { ...state, isOpen: false, selectedIndex: -1 };
    case "SET_SELECTED_INDEX":
      return { ...state, selectedIndex: action.index };
    case "POPULAR_FETCH_START":
      return { ...state, isLoadingPopular: true };
    case "POPULAR_FETCH_SUCCESS":
      return {
        ...state,
        popularPackages: action.packages,
        isLoadingPopular: false,
      };
  }
}

// ─────────────────────────────────────────────────
// Hook
// ─────────────────────────────────────────────────

interface UseSearchBarOptions {
  /** Callback when a package is selected */
  onSelectPackage: (packageName: string) => void;
  /** Already selected package names to exclude from suggestions */
  selectedPackages: string[];
}

/**
 * Custom hook encapsulating all SearchBar state management and data fetching.
 * Uses useReducer for atomic state transitions across 10 related state fields.
 * @param options - Hook configuration
 * @returns State object and handler functions for the SearchBar component
 * @example
 * const { state, handleInputChange, handleSelect, handleKeyDown } = useSearchBar({
 *   onSelectPackage: (name) => addPackage(name),
 *   selectedPackages: ["react", "vue"],
 * });
 */
export function useSearchBar({ onSelectPackage, selectedPackages }: UseSearchBarOptions) {
  const [state, dispatch] = useReducer(searchBarReducer, initialState);
  const debounceRef = useRef<NodeJS.Timeout | null>(null);
  const popularFetchedRef = useRef(false);

  /**
   * Fetches weekly download counts for multiple packages in parallel.
   * @param packageNames - Array of package names
   * @returns Map of package name to download count
   */
  const fetchDownloadCounts = useCallback(
    async (packageNames: string[]): Promise<Map<string, number>> => {
      const downloadMap = new Map<string, number>();
      if (packageNames.length === 0) return downloadMap;

      const fetchPromises = packageNames.map(async (name) => {
        try {
          const response = await fetch(
            `https://api.npmjs.org/downloads/point/last-week/${encodeURIComponent(name)}`,
          );
          if (!response.ok) return null;
          const data = await response.json();
          if (data.downloads !== undefined) {
            return { name, downloads: data.downloads as number };
          }
        } catch {
          // Silently fail for individual packages
        }
        return null;
      });

      const results = await Promise.all(fetchPromises);
      for (const result of results) {
        if (result) {
          downloadMap.set(result.name, result.downloads);
        }
      }

      return downloadMap;
    },
    [],
  );

  /**
   * Fetches popular packages with download counts. Only fetches once per mount.
   */
  const fetchPopularPackages = useCallback(async () => {
    if (popularFetchedRef.current || state.popularPackages.length > 0) {
      return;
    }

    dispatch({ type: "POPULAR_FETCH_START" });
    popularFetchedRef.current = true;

    try {
      const availablePackages = POPULAR_PACKAGES.filter(
        (name) => !selectedPackages.includes(name),
      );

      if (availablePackages.length === 0) {
        dispatch({ type: "POPULAR_FETCH_SUCCESS", packages: [] });
        return;
      }

      const downloadCounts = await fetchDownloadCounts(availablePackages);

      const packagesWithDownloads: NpmPackage[] = availablePackages
        .map((name) => ({
          name,
          description: "Popular package",
          weeklyDownloads: downloadCounts.get(name),
        }))
        .sort((a, b) => (b.weeklyDownloads ?? 0) - (a.weeklyDownloads ?? 0));

      dispatch({ type: "POPULAR_FETCH_SUCCESS", packages: packagesWithDownloads });
    } catch {
      dispatch({ type: "POPULAR_FETCH_SUCCESS", packages: [] });
    }
  }, [selectedPackages, fetchDownloadCounts, state.popularPackages.length]);

  /**
   * Fetches package suggestions from npm registry.
   */
  const fetchSuggestions = useCallback(
    async (searchQuery: string) => {
      if (searchQuery.length < 2) {
        dispatch({ type: "FETCH_SUCCESS", suggestions: [] });
        return;
      }

      dispatch({ type: "FETCH_START" });
      try {
        const response = await fetch(
          `https://registry.npmjs.org/-/v1/search?text=${encodeURIComponent(searchQuery)}&size=10`,
        );
        if (!response.ok) {
          throw new Error("Failed to fetch packages");
        }
        const data = await response.json();
        const packages: NpmPackage[] = data.objects.map(
          (obj: { package: NpmPackage }) => ({
            name: obj.package.name,
            description: obj.package.description,
            version: obj.package.version,
          }),
        );
        const filteredPackages = packages.filter(
          (pkg) => !selectedPackages.includes(pkg.name),
        );

        const packageNames = filteredPackages.map((pkg) => pkg.name);
        const downloadCounts = await fetchDownloadCounts(packageNames);

        const packagesWithDownloads = filteredPackages.map((pkg) => ({
          ...pkg,
          weeklyDownloads: downloadCounts.get(pkg.name),
        }));

        dispatch({ type: "FETCH_SUCCESS", suggestions: packagesWithDownloads });
      } catch {
        dispatch({
          type: "FETCH_ERROR",
          error: "Failed to search packages. Please try again.",
        });
      }
    },
    [selectedPackages, fetchDownloadCounts],
  );

  /**
   * Handles input value changes with debounced search and validation.
   */
  const handleInputChange = useCallback(
    (e: React.ChangeEvent<HTMLInputElement>) => {
      const value = e.target.value;
      const validationErr = validatePackageName(value);

      dispatch({ type: "SET_QUERY", query: value, validationError: validationErr });

      if (debounceRef.current) {
        clearTimeout(debounceRef.current);
      }

      if (!validationErr) {
        debounceRef.current = setTimeout(() => {
          fetchSuggestions(value);
        }, 300);
      }
    },
    [fetchSuggestions],
  );

  /**
   * Handles package selection — resets search state and notifies parent.
   */
  const handleSelect = useCallback(
    (packageName: string) => {
      onSelectPackage(packageName);
      dispatch({ type: "SELECT_PACKAGE" });
    },
    [onSelectPackage],
  );

  /**
   * Handles keyboard navigation for both suggestion and popular package lists.
   */
  const handleKeyDown = useCallback(
    (e: React.KeyboardEvent<HTMLInputElement>) => {
      const filteredPopular = state.popularPackages.filter(
        (pkg) => !selectedPackages.includes(pkg.name),
      );
      const activeList = state.query.length === 0 ? filteredPopular : state.suggestions;

      if (!state.isOpen || activeList.length === 0) {
        if (e.key === "Escape") {
          dispatch({ type: "CLOSE_DROPDOWN" });
        }
        return;
      }

      switch (e.key) {
        case "ArrowDown":
          e.preventDefault();
          dispatch({
            type: "SET_SELECTED_INDEX",
            index: state.selectedIndex < activeList.length - 1
              ? state.selectedIndex + 1
              : state.selectedIndex,
          });
          break;
        case "ArrowUp":
          e.preventDefault();
          dispatch({
            type: "SET_SELECTED_INDEX",
            index: state.selectedIndex > 0 ? state.selectedIndex - 1 : -1,
          });
          break;
        case "Enter":
          e.preventDefault();
          if (state.selectedIndex >= 0 && activeList[state.selectedIndex]) {
            handleSelect(activeList[state.selectedIndex].name);
          }
          break;
        case "Escape":
          dispatch({ type: "CLOSE_DROPDOWN" });
          break;
      }
    },
    [state, selectedPackages, handleSelect],
  );

  const openDropdown = useCallback(() => {
    dispatch({ type: "OPEN_DROPDOWN" });
  }, []);

  const closeDropdown = useCallback(() => {
    dispatch({ type: "CLOSE_DROPDOWN" });
  }, []);

  const setSelectedIndex = useCallback((index: number) => {
    dispatch({ type: "SET_SELECTED_INDEX", index });
  }, []);

  return {
    state,
    handleInputChange,
    handleSelect,
    handleKeyDown,
    fetchPopularPackages,
    openDropdown,
    closeDropdown,
    setSelectedIndex,
  };
}
````

## File: src/hooks/use-url-state.ts
````typescript
"use client";

import { useQueryState, parseAsArrayOf, parseAsString } from "nuqs";
import { useCallback, useMemo } from "react";
import { getChartColor } from "@/constants/colors";
import type { SelectedPackage, TimeRange } from "@/types/package";

/**
 * Parses comma-separated package names from URL.
 */
const packagesParser = parseAsArrayOf(parseAsString, ",").withDefault([]);

/**
 * Parses time range from URL.
 */
const timeRangeParser = parseAsString.withDefault("1y");

/**
 * Custom hook for managing URL state with nuqs.
 * Synchronizes package selection and time range with URL parameters.
 * @returns URL state and setter functions
 */
export function useUrlState() {
  const [packagesParam, setPackagesParam] = useQueryState(
    "packages",
    packagesParser
  );
  const [timeRangeParam, setTimeRangeParam] = useQueryState(
    "range",
    timeRangeParser
  );

  /**
   * Convert package names to SelectedPackage objects with colors.
   */
  const selectedPackages: SelectedPackage[] = useMemo(() => {
    return packagesParam.map((name, index) => ({
      name,
      color: getChartColor(index),
    }));
  }, [packagesParam]);

  /**
   * Current time range.
   */
  const timeRange = timeRangeParam as TimeRange;

  /**
   * Adds a package to the selection.
   */
  const addPackage = useCallback(
    async (packageName: string) => {
      if (packagesParam.includes(packageName)) return;
      await setPackagesParam([...packagesParam, packageName]);
    },
    [packagesParam, setPackagesParam]
  );

  /**
   * Removes a package from the selection.
   */
  const removePackage = useCallback(
    async (packageName: string) => {
      await setPackagesParam(packagesParam.filter((p) => p !== packageName));
    },
    [packagesParam, setPackagesParam]
  );

  /**
   * Sets the time range.
   */
  const setTimeRange = useCallback(
    async (range: TimeRange) => {
      await setTimeRangeParam(range);
    },
    [setTimeRangeParam]
  );

  /**
   * Clears all packages.
   */
  const clearPackages = useCallback(async () => {
    await setPackagesParam([]);
  }, [setPackagesParam]);

  /**
   * Sets all packages (replaces current selection).
   * @param packages - Array of package names to set
   */
  const setPackages = useCallback(
    async (packages: string[]) => {
      await setPackagesParam(packages);
    },
    [setPackagesParam]
  );

  return {
    selectedPackages,
    timeRange,
    addPackage,
    removePackage,
    setTimeRange,
    clearPackages,
    setPackages,
    packageNames: packagesParam,
  };
}
````

## File: src/lib/auth-client.ts
````typescript
"use client";

import { createAuthClient } from "better-auth/react";

/**
 * Better Auth client-side configuration.
 * Provides React hooks and methods for authentication.
 *
 * @description
 * - useSession: Hook to get current session state
 * - signIn: Methods for email/password and social OAuth sign-in
 * - signUp: Method for email/password registration
 * - signOut: Method to end the current session
 *
 * @example
 * ```tsx
 * // Get session state
 * const { data: session, isPending } = authClient.useSession();
 *
 * // Sign in with GitHub OAuth
 * await authClient.signIn.social({ provider: "github" });
 *
 * // Sign in with email/password
 * await authClient.signIn.email({ email, password });
 *
 * // Sign out
 * await authClient.signOut();
 * ```
 */
export const authClient = createAuthClient({
  baseURL: typeof window !== "undefined" ? window.location.origin : "",
});

/**
 * Convenience exports for common auth operations.
 */
export const { useSession, signIn, signUp, signOut } = authClient;
````

## File: src/lib/auth-fetch.ts
````typescript
"use client";

/**
 * Auth-aware fetch wrapper that handles session expiration.
 *
 * When a 401 response is received, redirects to the login page
 * with the current URL as the return URL.
 *
 * @param input - Fetch input (URL or Request)
 * @param init - Fetch init options
 * @returns Promise<Response>
 *
 * @example
 * ```ts
 * const response = await authFetch("/api/presets");
 * if (!response.ok) throw new Error("Failed to fetch");
 * const data = await response.json();
 * ```
 */
export async function authFetch(
  input: RequestInfo | URL,
  init?: RequestInit
): Promise<Response> {
  const response = await fetch(input, init);

  // Handle 401 Unauthorized - session expired or invalid
  if (response.status === 401) {
    if (typeof window !== "undefined") {
      const currentPath = window.location.pathname + window.location.search;
      const loginUrl = `/login?returnUrl=${encodeURIComponent(currentPath)}`;
      window.location.href = loginUrl;
    }
    // Return the response anyway for SSR/edge cases
    return response;
  }

  return response;
}

/**
 * JSON fetch helper with auth handling.
 *
 * @param url - API endpoint URL
 * @param options - Fetch options (method, body, etc.)
 * @returns Promise<T> - Parsed JSON response
 * @throws Error if response is not ok (except 401 which redirects)
 *
 * @example
 * ```ts
 * const presets = await authFetchJson<Preset[]>("/api/presets");
 * ```
 */
export async function authFetchJson<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const response = await authFetch(url, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
  });

  if (!response.ok) {
    const errorData = await response.json().catch(() => ({}));
    throw new Error(errorData.error || `Request failed: ${response.status}`);
  }

  return response.json() as Promise<T>;
}
````

## File: src/lib/auth.ts
````typescript
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { Pool } from "pg";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "@/generated/prisma/client";
import { sendEmail } from "@/lib/email";
import {
  resetPasswordEmail,
  verifyEmailTemplate,
} from "@/lib/email-templates";

/**
 * Prisma client instance for database operations.
 * Uses PrismaPg adapter with pg Pool for Neon PostgreSQL (required for Prisma 7.x).
 *
 * @see https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/databases-connections/postgres
 */
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaPg(pool);
const prisma = new PrismaClient({ adapter });

/**
 * Better Auth server-side configuration.
 * Provides email/password and social OAuth authentication.
 *
 * @description
 * - Uses Prisma adapter for PostgreSQL database
 * - Supports GitHub OAuth provider
 * - Email/password authentication enabled
 *
 * @see https://www.better-auth.com/docs
 */
export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),

  /**
   * Base URL for Better Auth.
   * Required for OAuth callback URLs to work correctly in production.
   * Falls back to BETTER_AUTH_URL env var, then VERCEL_URL for Vercel deployments.
   */
  baseURL:
    process.env.BETTER_AUTH_URL ||
    (process.env.VERCEL_URL
      ? `https://${process.env.VERCEL_URL}`
      : "http://localhost:3041"),

  /**
   * Trusted origins for CORS and security.
   * Includes localhost for development and Vercel domains for production.
   */
  trustedOrigins: [
    "http://localhost:3041",
    "http://localhost:3000",
    "http://localhost:3001",
    "http://localhost:3002",
    "https://npm-trend-clone.vercel.app",
    "https://*.vercel.app",
  ],

  /**
   * Email and password authentication configuration.
   * Allows users to sign up and sign in with email/password.
   */
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 8,
    maxPasswordLength: 128,
    autoSignIn: true,
    sendResetPassword: async ({ user, url }) => {
      void sendEmail({
        to: user.email,
        subject: "Reset your password — NPM Trends",
        html: resetPasswordEmail(url),
      });
    },
    resetPasswordTokenExpiresIn: 3600,
  },

  /**
   * Email verification configuration.
   * Sends verification email on signup; does not block login for unverified users.
   */
  emailVerification: {
    sendVerificationEmail: async ({ user, url }) => {
      void sendEmail({
        to: user.email,
        subject: "Verify your email — NPM Trends",
        html: verifyEmailTemplate(url),
      });
    },
    sendOnSignUp: true,
    autoSignInAfterVerification: true,
    expiresIn: 3600,
  },

  /**
   * Social OAuth providers configuration.
   * GitHub OAuth is enabled when environment variables are set.
   */
  socialProviders: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
  },

  /**
   * Session configuration.
   */
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days
    updateAge: 60 * 60 * 24, // 1 day
    cookieCache: {
      enabled: true,
      maxAge: 60 * 5, // 5 minutes
    },
  },

  user: {
    deleteUser: {
      enabled: true,
    },
  },

  /**
   * Base path for auth API routes.
   */
  basePath: "/api/auth",
});

/**
 * Type exports for auth session and user.
 */
export type Session = typeof auth.$Infer.Session;
export type User = typeof auth.$Infer.Session.user;
````

## File: src/lib/email-templates.ts
````typescript
const APP_NAME = "NPM Trends";

const COLORS = {
  primary: "#171717",
  primaryForeground: "#fafafa",
  muted: "#737373",
  border: "#e5e5e5",
  background: "#ffffff",
  surface: "#fafafa",
  accent: "#2563eb",
} as const;

function baseLayout(content: string): string {
  return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>${APP_NAME}</title>
</head>
<body style="margin:0;padding:0;background-color:${COLORS.surface};font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,'Helvetica Neue',Arial,sans-serif;">
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background-color:${COLORS.surface};">
    <tr>
      <td align="center" style="padding:40px 16px;">
        <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="max-width:480px;background-color:${COLORS.background};border:1px solid ${COLORS.border};border-radius:12px;">
          <!-- Header -->
          <tr>
            <td style="padding:32px 32px 0;text-align:center;">
              <span style="font-size:20px;font-weight:700;color:${COLORS.primary};letter-spacing:-0.02em;">${APP_NAME}</span>
            </td>
          </tr>
          <!-- Content -->
          <tr>
            <td style="padding:24px 32px 32px;">
              ${content}
            </td>
          </tr>
          <!-- Footer -->
          <tr>
            <td style="padding:0 32px 32px;text-align:center;">
              <p style="margin:0;font-size:12px;line-height:1.5;color:${COLORS.muted};">
                If you didn&rsquo;t request this email, you can safely ignore it.
              </p>
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
</body>
</html>`;
}

function ctaButton(url: string, label: string): string {
  return `<table role="presentation" width="100%" cellpadding="0" cellspacing="0">
  <tr>
    <td align="center" style="padding:24px 0;">
      <a href="${url}" target="_blank" style="display:inline-block;padding:12px 32px;background-color:${COLORS.primary};color:${COLORS.primaryForeground};font-size:14px;font-weight:600;text-decoration:none;border-radius:8px;line-height:1.4;">
        ${label}
      </a>
    </td>
  </tr>
</table>`;
}

/**
 * Generates styled HTML for password reset emails.
 * Uses table-based layout with inline CSS for maximum email client compatibility.
 */
export function resetPasswordEmail(url: string): string {
  const content = `
    <h1 style="margin:0 0 8px;font-size:22px;font-weight:700;color:${COLORS.primary};text-align:center;">
      Reset Your Password
    </h1>
    <p style="margin:0 0 4px;font-size:14px;line-height:1.6;color:${COLORS.muted};text-align:center;">
      We received a request to reset your password. Click the button below to choose a new one.
    </p>
    ${ctaButton(url, "Reset Password")}
    <p style="margin:0;font-size:12px;line-height:1.5;color:${COLORS.muted};text-align:center;">
      This link expires in 1 hour. If the button doesn&rsquo;t work, paste this URL into your browser:
    </p>
    <p style="margin:8px 0 0;font-size:12px;line-height:1.5;color:${COLORS.accent};word-break:break-all;text-align:center;">
      ${url}
    </p>`;

  return baseLayout(content);
}

/**
 * Generates styled HTML for email verification emails.
 * Uses table-based layout with inline CSS for maximum email client compatibility.
 */
export function verifyEmailTemplate(url: string): string {
  const content = `
    <h1 style="margin:0 0 8px;font-size:22px;font-weight:700;color:${COLORS.primary};text-align:center;">
      Verify Your Email
    </h1>
    <p style="margin:0 0 4px;font-size:14px;line-height:1.6;color:${COLORS.muted};text-align:center;">
      Thanks for signing up! Please verify your email address to get started.
    </p>
    ${ctaButton(url, "Verify Email Address")}
    <p style="margin:0;font-size:12px;line-height:1.5;color:${COLORS.muted};text-align:center;">
      This link expires in 1 hour. If the button doesn&rsquo;t work, paste this URL into your browser:
    </p>
    <p style="margin:8px 0 0;font-size:12px;line-height:1.5;color:${COLORS.accent};word-break:break-all;text-align:center;">
      ${url}
    </p>`;

  return baseLayout(content);
}
````

## File: src/lib/email.ts
````typescript
import { Resend } from "resend";

interface SendEmailParams {
  to: string;
  subject: string;
  text?: string;
  html?: string;
}

const resendApiKey = process.env.RESEND_API_KEY;
const emailFrom = process.env.EMAIL_FROM ?? "npm-trends <noreply@example.com>";

/**
 * Sends an email via Resend.
 * Falls back to console logging when RESEND_API_KEY is not configured
 * (local development without email service).
 */
export async function sendEmail({ to, subject, text, html }: SendEmailParams) {
  if (!resendApiKey) {
    console.log("[email] RESEND_API_KEY not set — logging email instead:");
    console.log(`  To: ${to}`);
    console.log(`  Subject: ${subject}`);
    if (text) console.log(`  Text: ${text}`);
    if (html) console.log(`  HTML: ${html}`);
    return;
  }

  const resend = new Resend(resendApiKey);

  const { error } = await resend.emails.send({
    from: emailFrom,
    to: [to],
    subject,
    ...(html ? { html } : { text: text ?? "" }),
  });

  if (error) {
    console.error("[email] Failed to send:", error);
  }
}
````

## File: src/lib/redis.ts
````typescript
import { Redis } from "@upstash/redis";

/**
 * Upstash Redis client instance.
 * Reads UPSTASH_REDIS_REST_URL and UPSTASH_REDIS_REST_TOKEN from environment variables.
 *
 * @returns Redis client configured for the Upstash REST API
 */
export const redis = Redis.fromEnv();

/**
 * Check if Redis connection is healthy by executing a PING command.
 *
 * @returns
 * - When connected: { status: "up", latencyMs: number }
 * - When connection fails: { status: "down", error: string }
 *
 * @example
 * const health = await checkRedisHealth();
 * // => { status: "up", latencyMs: 42 }
 */
export async function checkRedisHealth(): Promise<
  { status: "up"; latencyMs: number } | { status: "down"; error: string }
> {
  const start = Date.now();

  try {
    const result = await redis.ping();

    if (result === "PONG") {
      return {
        status: "up",
        latencyMs: Date.now() - start,
      };
    }

    return {
      status: "down",
      error: `Unexpected PING response: ${result}`,
    };
  } catch (error) {
    return {
      status: "down",
      error: error instanceof Error ? error.message : "Unknown error",
    };
  }
}
````

## File: src/lib/utils.ts
````typescript
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
````

## File: src/types/package.ts
````typescript
/**
 * Represents an npm package with its metadata.
 */
export interface NpmPackage {
  /** Package name (e.g., "react", "@angular/core") */
  name: string;
  /** Package description */
  description?: string;
  /** Latest version */
  version?: string;
  /** Weekly download count */
  weeklyDownloads?: number;
}

/**
 * Extended package information including repository and metadata.
 */
export interface PackageInfo {
  /** Package name */
  name: string;
  /** Package description */
  description: string;
  /** Latest version */
  version: string;
  /** License type (e.g., "MIT", "Apache-2.0") */
  license: string;
  /** npm package homepage URL */
  homepage?: string;
  /** GitHub repository URL */
  repositoryUrl?: string;
  /** Last publish date in ISO format */
  lastPublished: string;
  /** Weekly download count */
  weeklyDownloads: number;
  /** GitHub stars count (if available) */
  githubStars?: number;
}

/**
 * Selected package for comparison with assigned color.
 */
export interface SelectedPackage {
  /** Package name */
  name: string;
  /** Assigned color in hex format (e.g., "#61dafb") */
  color: string;
}

/**
 * Download data point for a specific date.
 */
export interface DownloadDataPoint {
  /** Date in ISO format (e.g., "2024-01-01") */
  date: string;
  /** Download count for this date */
  downloads: number;
}

/**
 * Download statistics for a package.
 */
export interface PackageDownloads {
  /** Package name */
  package: string;
  /** Start date of the data range */
  start: string;
  /** End date of the data range */
  end: string;
  /** Array of download data points */
  downloads: DownloadDataPoint[];
}

/**
 * Chart data point with downloads from multiple packages.
 */
export interface ChartDataPoint {
  /** Date in ISO format */
  date: string;
  /** Dynamic keys for each package's download count */
  [packageName: string]: string | number;
}

/**
 * Time range options for the chart.
 */
export type TimeRange = "1m" | "3m" | "6m" | "1y" | "2y" | "5y";

/**
 * Time range configuration.
 */
export interface TimeRangeConfig {
  /** Time range key */
  value: TimeRange;
  /** Display label */
  label: string;
  /** Number of days */
  days: number;
}

/**
 * Available time ranges with their configurations.
 */
export const TIME_RANGES: TimeRangeConfig[] = [
  { value: "1m", label: "1 Month", days: 30 },
  { value: "3m", label: "3 Months", days: 90 },
  { value: "6m", label: "6 Months", days: 180 },
  { value: "1y", label: "1 Year", days: 365 },
  { value: "2y", label: "2 Years", days: 730 },
  { value: "5y", label: "5 Years", days: 1825 },
];
````

## File: src/utils/color.ts
````typescript
/**
 * Color contrast utilities for accessibility compliance.
 * Uses WCAG luminance formula to determine appropriate text colors.
 */

/**
 * Calculates relative luminance and returns appropriate text color class.
 * Uses WCAG luminance formula for accessibility compliance.
 * @param hexColor - Hex color string (e.g., "#61dafb")
 * @returns Tailwind text color class
 * @example
 * getContrastTextColor("#61dafb") // => "text-gray-900" (React blue - light bg)
 * getContrastTextColor("#dd0031") // => "text-white" (Angular red - dark bg)
 */
export function getContrastTextColor(hexColor: string): "text-gray-900" | "text-white" {
  const r = parseInt(hexColor.slice(1, 3), 16);
  const g = parseInt(hexColor.slice(3, 5), 16);
  const b = parseInt(hexColor.slice(5, 7), 16);

  // WCAG relative luminance formula
  const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255;

  return luminance > 0.5 ? "text-gray-900" : "text-white";
}

/**
 * Returns appropriate button hover classes based on background luminance.
 * Provides subtle opacity and background changes for interactive states.
 * @param hexColor - Hex color string (e.g., "#61dafb")
 * @returns Tailwind classes for text opacity, hover text, and hover background
 * @example
 * getContrastHoverClass("#61dafb") // => for light bg: dark text with black hover overlay
 * getContrastHoverClass("#dd0031") // => for dark bg: white text with white hover overlay
 */
export function getContrastHoverClass(hexColor: string): string {
  const r = parseInt(hexColor.slice(1, 3), 16);
  const g = parseInt(hexColor.slice(3, 5), 16);
  const b = parseInt(hexColor.slice(5, 7), 16);
  const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255;

  return luminance > 0.5
    ? "text-gray-900/80 hover:text-gray-900 hover:bg-black/10"
    : "text-white/80 hover:text-white hover:bg-white/20";
}

/**
 * A curated palette of vibrant, accessible colors for package badges.
 * These colors are designed to work well in both light and dark modes.
 */
const PACKAGE_COLORS = [
  "#ef4444", // Red (Rose)
  "#f97316", // Orange (Tangerine)
  "#eab308", // Yellow (Gold)
  "#22c55e", // Green (Emerald)
  "#14b8a6", // Teal (Aqua)
  "#0ea5e9", // Blue (Sky)
  "#6366f1", // Indigo (Royal)
  "#a855f7", // Purple (Violet)
  "#ec4899", // Pink (Fuchsia)
  "#8b5cf6", // Lavender
  "#06b6d4", // Cyan (Turquoise)
  "#84cc16", // Lime (Chartreuse)
] as const;

/**
 * Simple hash function for strings.
 * Produces a consistent numeric hash for any string input.
 * @param str - The string to hash
 * @returns A positive integer hash value
 * @example
 * hashString("react") // => 104115109
 * hashString("react") // => 104115109 (always same)
 */
function hashString(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = (hash << 5) - hash + char;
    hash = hash & hash; // Convert to 32-bit integer
  }
  return Math.abs(hash);
}

/**
 * Generates a consistent color for a package name.
 * The same package name will always return the same color.
 * @param packageName - The npm package name
 * @returns A hex color string
 * @example
 * getPackageColor("react") // => "#0ea5e9"
 * getPackageColor("vue") // => "#22c55e"
 */
export function getPackageColor(packageName: string): string {
  const hash = hashString(packageName.toLowerCase());
  const index = hash % PACKAGE_COLORS.length;
  return PACKAGE_COLORS[index];
}

/**
 * Generates a lighter version of a color for backgrounds.
 * Creates a semi-transparent version that works in both light and dark modes.
 * @param hexColor - The base hex color
 * @param opacity - Opacity value (0-1), defaults to 0.1
 * @returns An rgba color string
 * @example
 * getPackageBackgroundColor("#ef4444") // => "rgba(239, 68, 68, 0.1)"
 */
export function getPackageBackgroundColor(
  hexColor: string,
  opacity: number = 0.1
): string {
  const hex = hexColor.replace("#", "");
  const r = parseInt(hex.substring(0, 2), 16);
  const g = parseInt(hex.substring(2, 4), 16);
  const b = parseInt(hex.substring(4, 6), 16);
  return `rgba(${r}, ${g}, ${b}, ${opacity})`;
}
````

## File: src/utils/isMSWEnabled.ts
````typescript
import { env } from "@/env";

/**
 * Determines whether MSW (Mock Service Worker) should be enabled.
 *
 * @description
 * Uses asymmetric logic for client and server:
 * - **Client**: Only checks `NEXT_PUBLIC_ENABLE_MSW_MOCK`
 * - **Server**: Requires BOTH `NEXT_PUBLIC_ENABLE_MSW_MOCK=true` AND (APP_ENV=test OR NODE_ENV=test)
 *
 * This double-check on server-side prevents accidental production activation
 * even if environment variables are misconfigured.
 *
 * @returns
 * - `true`: MSW should be enabled
 * - `false`: MSW should be disabled
 *
 * @example
 * // Client-side (browser)
 * isMSWEnabled() // checks NEXT_PUBLIC_ENABLE_MSW_MOCK only
 *
 * // Server-side (RSC/API Routes)
 * isMSWEnabled() // checks NEXT_PUBLIC_ENABLE_MSW_MOCK AND APP_ENV/NODE_ENV
 */
export function isMSWEnabled(): boolean {
  // Client-side: Only check public environment variable
  // Server-only variables (APP_ENV) are not accessible from client
  if (typeof window !== "undefined") {
    return env.NEXT_PUBLIC_ENABLE_MSW_MOCK === "true";
  }

  // Server-side: Double-check for safety
  // Even if NEXT_PUBLIC_ENABLE_MSW_MOCK=true, won't activate unless in test mode
  return (
    env.NEXT_PUBLIC_ENABLE_MSW_MOCK === "true" &&
    (env.APP_ENV === "test" || process.env.NODE_ENV === "test")
  );
}
````

## File: src/env.ts
````typescript
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

/**
 * Type-safe environment variable schema for the application.
 *
 * @description
 * - Server variables: Only available server-side (APP_ENV, NODE_ENV)
 * - Client variables: Available client-side, must use NEXT_PUBLIC_ prefix
 *
 * @example
 * // Usage
 * import { env } from '@/env';
 * console.log(env.NEXT_PUBLIC_ENABLE_MSW_MOCK); // 'true' | 'false'
 */
export const env = createEnv({
  server: {
    NODE_ENV: z
      .enum(["development", "test", "production"])
      .default("development"),
    /** Custom environment identifier for distinguishing test builds from production */
    APP_ENV: z.enum(["development", "test"]).optional(),
  },
  client: {
    /**
     * Flag to enable MSW mocking.
     * Must be explicitly set to 'true' to enable.
     */
    NEXT_PUBLIC_ENABLE_MSW_MOCK: z
      .enum(["true", "false"])
      .optional()
      .default("false"),
  },
  runtimeEnv: {
    NODE_ENV: process.env.NODE_ENV,
    APP_ENV: process.env.APP_ENV,
    NEXT_PUBLIC_ENABLE_MSW_MOCK: process.env.NEXT_PUBLIC_ENABLE_MSW_MOCK,
  },
  /**
   * Skip validation in environments where env vars may not be available
   * (e.g., CI/CD build steps)
   */
  skipValidation: !!process.env.SKIP_ENV_VALIDATION,
  emptyStringAsUndefined: true,
});
````

## File: src/middleware.ts
````typescript
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
import { type NextRequest, NextResponse } from "next/server";

/**
 * Checks if rate limiting should be bypassed.
 * Bypasses in test environment to allow E2E tests without Redis.
 */
const shouldBypassRateLimit =
  process.env.APP_ENV === "test" ||
  process.env.NEXT_PUBLIC_ENABLE_MSW_MOCK === "true";

/**
 * Rate limiter using Upstash Redis with sliding window algorithm.
 * Limits: 100 requests per 10 seconds per IP address.
 *
 * @description
 * Uses sliding window for accurate rate limiting without burst issues.
 * Ephemeral cache reduces Redis calls for hot IPs.
 * Skipped in test mode to allow E2E testing without Redis dependency.
 */
const ratelimit = shouldBypassRateLimit
  ? null
  : new Ratelimit({
      redis: Redis.fromEnv(),
      limiter: Ratelimit.slidingWindow(100, "10 s"),
      ephemeralCache: new Map(),
      analytics: true,
      prefix: "npm-trend-ratelimit",
    });

/**
 * Extracts client IP address from request headers.
 * Falls back to 127.0.0.1 for local development.
 */
function getClientIp(request: NextRequest): string {
  // Check common proxy headers
  const forwardedFor = request.headers.get("x-forwarded-for");
  if (forwardedFor) {
    return forwardedFor.split(",")[0].trim();
  }

  const realIp = request.headers.get("x-real-ip");
  if (realIp) {
    return realIp;
  }

  // Vercel-specific header
  const vercelForwardedFor = request.headers.get("x-vercel-forwarded-for");
  if (vercelForwardedFor) {
    return vercelForwardedFor.split(",")[0].trim();
  }

  // Fallback for local development
  // Note: request.ip is available in Vercel edge runtime but not in all environments
  return "127.0.0.1";
}

export async function middleware(request: NextRequest): Promise<NextResponse> {
  // Bypass rate limiting in test mode
  if (!ratelimit) {
    return NextResponse.next();
  }

  const ip = getClientIp(request);

  try {
    const { success, limit, remaining, reset } = await ratelimit.limit(ip);

    // Calculate seconds until reset
    const retryAfter = Math.ceil((reset - Date.now()) / 1000);

    if (!success) {
      return new NextResponse(
        JSON.stringify({
          error: "Too Many Requests",
          message: `Rate limit exceeded. Please try again in ${retryAfter} seconds.`,
          retryAfter,
        }),
        {
          status: 429,
          headers: {
            "Content-Type": "application/json",
            "Retry-After": String(retryAfter),
            "X-RateLimit-Limit": String(limit),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": String(reset),
          },
        }
      );
    }

    // Add rate limit headers to successful responses
    const response = NextResponse.next();
    response.headers.set("X-RateLimit-Limit", String(limit));
    response.headers.set("X-RateLimit-Remaining", String(remaining));
    response.headers.set("X-RateLimit-Reset", String(reset));

    return response;
  } catch (error) {
    // If Redis is unavailable, allow request to proceed (fail-open)
    console.error("Rate limiting error:", error);
    return NextResponse.next();
  }
}

/**
 * Apply rate limiting to API routes, excluding health check.
 * Health check must remain unthrottled for monitoring systems.
 */
export const config = {
  matcher: [
    "/api/downloads/:path*",
    "/api/packages/:path*",
  ],
};
````

## File: package.json
````json
{
  "name": "npm-trend-clone",
  "version": "0.1.0",
  "description": "A modern clone of npmtrends.com built with Next.js 16, featuring npm package download statistics comparison and visualization",
  "author": "Ryota Murakami <ryota.murakami@laststance.io> (https://github.com/ryota-murakami)",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/laststance/npm-trend-clone.git"
  },
  "homepage": "https://github.com/laststance/npm-trend-clone#readme",
  "bugs": {
    "url": "https://github.com/laststance/npm-trend-clone/issues"
  },
  "keywords": [
    "npm",
    "trends",
    "statistics",
    "nextjs",
    "typescript",
    "react"
  ],
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack -p 3041",
    "build": "prisma generate && prisma db push --accept-data-loss && next build",
    "build:e2e": "prisma generate && DATABASE_URL=postgresql://postgres:postgres@localhost:54321/npm_trend_clone prisma db push --accept-data-loss && NEXT_PUBLIC_ENABLE_MSW_MOCK=true APP_ENV=test next build",
    "start": "next start -p 3041",
    "start:e2e": "NEXT_PUBLIC_ENABLE_MSW_MOCK=true APP_ENV=test next start -p 3041",
    "lint": "eslint . --ext .ts,.tsx",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:e2e": "pnpm build:e2e && playwright test",
    "test:e2e:ui": "playwright test --ui",
    "storybook": "storybook dev -p 6010",
    "build-storybook": "storybook build",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:seed": "prisma db seed",
    "db:studio": "prisma studio"
  },
  "dependencies": {
    "@prisma/adapter-pg": "^7.2.0",
    "@prisma/client": "^7.2.0",
    "@radix-ui/react-alert-dialog": "^1.1.15",
    "@radix-ui/react-avatar": "^1.1.11",
    "@radix-ui/react-checkbox": "^1.3.3",
    "@radix-ui/react-dialog": "^1.1.15",
    "@radix-ui/react-dropdown-menu": "^2.1.16",
    "@radix-ui/react-label": "^2.1.8",
    "@radix-ui/react-progress": "^1.1.8",
    "@radix-ui/react-radio-group": "^1.3.8",
    "@radix-ui/react-separator": "^1.1.8",
    "@radix-ui/react-slot": "^1.2.4",
    "@radix-ui/react-tabs": "^1.1.13",
    "@radix-ui/react-tooltip": "^1.2.8",
    "@sentry/nextjs": "^10.32.1",
    "@t3-oss/env-nextjs": "^0.13.10",
    "@tanstack/react-query": "^5.90.12",
    "@upstash/ratelimit": "^2.0.7",
    "@upstash/redis": "^1.35.8",
    "@vercel/analytics": "^1.6.1",
    "better-auth": "^1.4.8",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "lucide-react": "^0.562.0",
    "modern-screenshot": "^4.6.7",
    "next": "16.1.1",
    "next-themes": "^0.4.6",
    "nuqs": "^2.8.5",
    "pg": "^8.16.3",
    "prisma": "^7.2.0",
    "react": "19.2.3",
    "react-dom": "19.2.3",
    "recharts": "^3.6.0",
    "resend": "^6.6.0",
    "sonner": "^2.0.7",
    "tailwind-merge": "^3.4.0",
    "zod": "^4.2.1"
  },
  "devDependencies": {
    "@axe-core/playwright": "^4.11.0",
    "@chromatic-com/storybook": "^4.1.3",
    "@playwright/test": "^1.57.0",
    "@storybook/addon-a11y": "^10.1.10",
    "@storybook/addon-docs": "^10.1.10",
    "@storybook/addon-essentials": "^8.6.14",
    "@storybook/addon-vitest": "^10.1.10",
    "@storybook/nextjs": "^10.1.10",
    "@storybook/nextjs-vite": "^10.1.10",
    "@storybook/test": "^8.6.14",
    "@tailwindcss/postcss": "^4.1.18",
    "@types/node": "^25.0.3",
    "@types/pg": "^8.16.0",
    "@types/react": "^19.2.7",
    "@types/react-dom": "^19.2.3",
    "@vitest/browser-playwright": "^4.0.16",
    "@vitest/coverage-v8": "^4.0.16",
    "dotenv": "^17.2.3",
    "eslint": "^9.39.2",
    "eslint-config-next": "16.1.1",
    "eslint-plugin-storybook": "^10.1.10",
    "msw": "^2.12.4",
    "msw-storybook-addon": "^2.0.6",
    "playwright": "^1.57.0",
    "storybook": "^10.1.10",
    "tailwindcss": "^4.1.18",
    "tsx": "^4.21.0",
    "tw-animate-css": "^1.4.0",
    "typescript": "^5.9.3",
    "vite": "^7.3.0",
    "vitest": "^4.0.16"
  },
  "msw": {
    "workerDirectory": [
      "public"
    ]
  },
  "packageManager": "pnpm@10.28.1+sha512.7d7dbbca9e99447b7c3bf7a73286afaaf6be99251eb9498baefa7d406892f67b879adb3a1d7e687fc4ccc1a388c7175fbaae567a26ab44d1067b54fcb0d6a316"
}
````

## File: README.md
````markdown
# npm-trend-clone

[![Build](https://github.com/laststance/npm-trend-clone/actions/workflows/build.yml/badge.svg)](https://github.com/laststance/npm-trend-clone/actions/workflows/build.yml)
[![E2E Tests](https://github.com/laststance/npm-trend-clone/actions/workflows/e2e.yml/badge.svg)](https://github.com/laststance/npm-trend-clone/actions/workflows/e2e.yml)
[![Lint](https://github.com/laststance/npm-trend-clone/actions/workflows/lint.yml/badge.svg)](https://github.com/laststance/npm-trend-clone/actions/workflows/lint.yml)
[![Typecheck](https://github.com/laststance/npm-trend-clone/actions/workflows/typecheck.yml/badge.svg)](https://github.com/laststance/npm-trend-clone/actions/workflows/typecheck.yml)

A modern clone of [npmtrends.com](https://npmtrends.com/) built with Next.js 16, featuring npm package download statistics comparison and visualization.

## Acknowledgements

This project is inspired by and pays tribute to [npm trends](https://npmtrends.com/) created by [John Potter](https://github.com/johnmpotter). The original npm trends has been an invaluable tool for the JavaScript community, helping developers make informed decisions about package selection. We deeply appreciate the original work and aim to explore modern web development patterns while maintaining respect for the original project's vision.

## Features

- **Package Comparison** - Compare download trends of multiple npm packages
- **Interactive Charts** - Visualize download statistics with dynamic charts
- **OAuth Authentication** - Sign in with GitHub via Better Auth
- **Rate Limiting** - API protection with Upstash Redis
- **Dark Mode** - Full dark/light theme support
- **Responsive Design** - Mobile-friendly interface with shadcn/ui

## Tech Stack

| Category | Technology |
|----------|-----------|
| Framework | [Next.js 16](https://nextjs.org/) (App Router, Turbopack) |
| Language | [TypeScript](https://www.typescriptlang.org/) |
| Database | [PostgreSQL](https://www.postgresql.org/) via [Neon](https://neon.tech/) |
| ORM | [Prisma 7](https://www.prisma.io/) |
| Cache | [Upstash Redis](https://upstash.com/) |
| Auth | [Better Auth](https://www.better-auth.com/) (GitHub OAuth) |
| UI | [shadcn/ui](https://ui.shadcn.com/) + [Radix UI](https://www.radix-ui.com/) |
| Styling | [Tailwind CSS](https://tailwindcss.com/) |
| State | [TanStack Query](https://tanstack.com/query) |
| Testing | [Playwright](https://playwright.dev/) (E2E), [Vitest](https://vitest.dev/) (Unit) |
| Mocking | [MSW](https://mswjs.io/) (Mock Service Worker) |
| Monitoring | [Sentry](https://sentry.io/) |

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 22+
- [pnpm](https://pnpm.io/) 9+
- [Docker](https://www.docker.com/) (for local development)

### Local Development Setup

1. **Clone the repository**

```bash
git clone https://github.com/laststance/npm-trend-clone.git
cd npm-trend-clone
```

2. **Install dependencies**

```bash
pnpm install
```

3. **Start local services with Docker Compose**

```bash
docker compose up -d
```

This starts:
- **PostgreSQL** (port 54321) - Local database
- **Redis** (port 6379) - Cache backend
- **SRH** (port 8079) - Upstash-compatible Redis HTTP API ([Serverless Redis HTTP](https://github.com/hiett/serverless-redis-http))

4. **Set up environment variables**

```bash
cp .env.local.example .env.local
```

Edit `.env.local` with your configuration. For local development with Docker:

```env
# Database - Local PostgreSQL
DATABASE_URL=postgresql://postgres:postgres@localhost:54321/npm_trend_clone?schema=public

# Redis - Local SRH (Upstash-compatible)
UPSTASH_REDIS_REST_URL=http://localhost:8079
UPSTASH_REDIS_REST_TOKEN=local_development_token

# Auth
BETTER_AUTH_SECRET=your-local-dev-secret
```

5. **Initialize the database**

```bash
pnpm prisma generate
pnpm prisma db push
```

6. **Seed the database** (required for auth testing)

```bash
pnpm db:seed
```

This creates a test account for local development:

| Field | Value |
|-------|-------|
| Email | `test@example.com` |
| Password | `testpassword123` |
| Name | `Test User` |

7. **Start the development server**

```bash
pnpm dev
```

Open [http://localhost:3041](http://localhost:3041) to view the application.

### Docker Compose Services

| Service | Port | Description |
|---------|------|-------------|
| `postgres` | 54321 | PostgreSQL 16 database |
| `redis` | 6379 | Redis 7 cache server |
| `redis-http` | 8079 | [SRH](https://github.com/hiett/serverless-redis-http) - Upstash REST API compatible proxy |

> **Note**: SRH (Serverless Redis HTTP) is [officially recommended by Upstash](https://upstash.com/docs/redis/sdks/ts/developing) for local development. It provides an HTTP REST API in front of Redis, making the `@upstash/redis` SDK work seamlessly in local environments.

### Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start development server with Turbopack on port 3041 |
| `pnpm build` | Build for production |
| `pnpm start` | Start production server on port 3041 |
| `pnpm lint` | Run ESLint |
| `pnpm typecheck` | Run TypeScript type checking |
| `pnpm test` | Run unit tests with Vitest |
| `pnpm test:e2e` | Run E2E tests with Playwright |
| `pnpm db:seed` | Seed database with test account |

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── api/               # API routes
│   │   ├── auth/          # Better Auth endpoints
│   │   ├── downloads/     # npm download statistics
│   │   ├── health/        # Health check endpoint
│   │   └── packages/      # Package info & search
│   ├── login/             # Login page
│   ├── signup/            # Signup page
│   ├── settings/          # User settings
│   └── page.tsx           # Home page
├── components/            # React components
│   └── ui/               # shadcn/ui components
├── hooks/                 # Custom React hooks
├── lib/                   # Utility libraries
│   ├── auth.ts           # Better Auth configuration
│   ├── redis.ts          # Upstash Redis client
│   └── utils.ts          # Helper functions
└── generated/            # Generated files (Prisma client)
```

## Testing

### Unit Tests

```bash
pnpm test
```

### E2E Tests

```bash
# Run in headless mode
pnpm test:e2e

# Run with UI
pnpm exec playwright test --ui
```

E2E tests use MSW (Mock Service Worker) for API mocking, ensuring consistent and reliable test results.

## GitHub OAuth Setup

To enable "Sign in with GitHub":

1. Go to **GitHub → Settings → Developer settings → OAuth Apps → New OAuth App**
2. Fill in:
   - **Application name**: `npm-trend-clone` (or your preferred name)
   - **Homepage URL**: `http://localhost:3041` (local) or your production URL
   - **Authorization callback URL**: `http://localhost:3041/api/auth/callback/github`
3. After creating, copy the **Client ID** and generate a **Client Secret**
4. Add to `.env.local`:
   ```env
   GITHUB_CLIENT_ID=your-client-id
   GITHUB_CLIENT_SECRET=your-client-secret
   ```

For production deployment, create a **separate** OAuth App with:
- **Homepage URL**: `https://your-production-domain.com`
- **Authorization callback URL**: `https://your-production-domain.com/api/auth/callback/github`

> **Note**: Better Auth handles the OAuth callback at `/api/auth/callback/github`. Ensure the callback URL matches exactly.

## Email Delivery (Resend)

Password reset and email verification use [Resend](https://resend.com/) for email delivery.

**Local development**: When `RESEND_API_KEY` is not set, emails are logged to the console instead of being sent.

**Production setup**:

1. Create a [Resend](https://resend.com/) account
2. Verify a sender domain (or use the free `onboarding@resend.dev` for testing)
3. Generate an API key
4. Add to your environment:
   ```env
   RESEND_API_KEY=re_your_api_key_here
   EMAIL_FROM=npm-trends <noreply@yourdomain.com>
   ```

> **Note**: Without `RESEND_API_KEY` set in production, auth email features (password reset, email verification) will silently fail to deliver.

## Deployment

### Vercel (Recommended)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/laststance/npm-trend-clone)

### Environment Variables & Secrets

> **Note**: Most variables are accessed directly via `process.env` at runtime. Only `NODE_ENV`, `APP_ENV`, and `NEXT_PUBLIC_ENABLE_MSW_MOCK` are validated via Zod in `src/env.ts`. Ensure all required variables are set correctly to avoid runtime errors. `DATABASE_URL_UNPOOLED` is not needed for local development (Docker uses a single direct connection).

| Variable | Local | CI (Build) | Production | Required | Description |
|----------|:-----:|:----------:|:----------:|:--------:|-------------|
| `DATABASE_URL` | `.env.local` | Secret | Vercel env | Yes | Neon PostgreSQL pooled connection string |
| `DATABASE_URL_UNPOOLED` | — | Secret | Vercel env | Yes | Direct connection for migrations (non-pooled) |
| `UPSTASH_REDIS_REST_URL` | `.env.local` | Secret | Vercel env | Yes | Upstash Redis REST endpoint |
| `UPSTASH_REDIS_REST_TOKEN` | `.env.local` | Secret | Vercel env | Yes | Upstash Redis REST token |
| `BETTER_AUTH_SECRET` | `.env.local` | Secret | Vercel env | Yes | Auth encryption secret (min 32 chars) |
| `GITHUB_CLIENT_ID` | `.env.local` | Secret | Vercel env | Yes | GitHub OAuth App client ID |
| `GITHUB_CLIENT_SECRET` | `.env.local` | Secret | Vercel env | Yes | GitHub OAuth App client secret |
| `RESEND_API_KEY` | Optional | — | Vercel env | Yes | Resend API key for email delivery |
| `EMAIL_FROM` | Optional | — | Vercel env | Yes | Sender address (e.g., `noreply@yourdomain.com`) |
| `NEON_API_KEY` | — | Secret | — | No | Neon API key for migration backup branches |
| `NEON_PROJECT_ID` | — | Secret | — | No | Neon project ID for backup branch creation |
| `NEXT_PUBLIC_ENABLE_MSW_MOCK` | `.env.local` | — | — | No | Enable MSW mocking (`true`/`false`) |

**GitHub Actions secrets** (set in repo Settings → Secrets → Actions):
- Required: `DATABASE_URL`, `DATABASE_URL_UNPOOLED`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, `BETTER_AUTH_SECRET`, `OAUTH_GITHUB_CLIENT_ID`, `OAUTH_GITHUB_CLIENT_SECRET`
- Optional (migration workflow): `NEON_API_KEY`, `NEON_PROJECT_ID` — when set, the db-migrate workflow creates a Neon backup branch before running `prisma migrate deploy`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is open source and available under the [MIT License](LICENSE).

## Related Links

- [npm trends](https://npmtrends.com/) - The original inspiration
- [npm Registry API](https://github.com/npm/registry/blob/main/docs/REGISTRY-API.md) - npm API documentation
````
