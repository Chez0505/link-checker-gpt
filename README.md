# Link-Checker GPT

Welcome to the Link-Checker GPT! Crafted with the assistance of ChatGPT, this link checker ensures the integrity of links in your website's content. Although primarily designed for the FluxCD website's preview environments, it's versatile enough to work with most platforms, including Netlify.

## Getting Started

To check the links of a preview environment on Netlify, simply run:

```bash
ruby main.rb deploy-preview-1573--fluxcd.netlify.app
```

When executed against the main `fluxcd.io` site and its preview version, the behavior should be consistent. Any discrepancies are considered bugs—either they're rectified in this tool or addressed directly in the website by modifying the links.

Upon successful execution, a report detailing the link statuses is generated in `report.csv`. You can import this CSV into tools like Google Drive for further analysis and action.

## Integration as a CI Check

Link-Checker GPT is ready to be integrated as a CI check within the fluxcd/website repository. When a PR check flags an error, it's an invitation to refine your links. An associated report is available as a downloadable CSV to guide the necessary corrections. Our bot might also add a comment to your PR, providing insights and recommendations to enhance the quality of your links.

Certainly! Below is an example integration guide that you can include in the README for users:

---

## Integration Guide for `fluxcd/website`

Integrating the Link-Checker GPT into your existing workflow is straightforward. Here's how you can integrate it into the `fluxcd/website` repository:

### Step 1: Add the Action

In your `.github/workflows/` directory (create it if it doesn't exist), add a new workflow file, for instance, `link-check.yml`.

Within this file, add the following content:

```yaml
name: Link Checker

on:
  pull_request:
    branches:
      - main

jobs:
  check-links:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Link Checker GPT
      uses: kingdonb/link-checker-gpt@v1
      with:
        prNumber: ${{ github.event.pull_request.number }}
```

Replace `your-repo` with the repository path where you've hosted the Link-Checker GPT action.

**TODO**: make this work for other consumers besides fluxcd.io

### Step 2: Configuration

The only required parameter for the action is `prNumber`, which is automatically extracted from the GitHub context for the pull request event. This is used to generate the preview URL for the link checker.

### Step 3: Commit and Test

Commit the new workflow file and create a new pull request. The Link Checker GPT action should automatically run and validate the links within the website content associated with the PR.

## Note on UX: Report Download

In the event of a PR check failure, you can read the report in the failed job output. Initially this workflow was designed to enable the user to access a detailed report in the form of a zipped CSV. You can still reuse the action as a composite workflow, but it is not the intended use anymore.

Instead, the report now goes out to the workflow/action job log. You can read all the bad links created by your PR there. Any links from the baseline site will not be included in the report unless your PR is spotless. Then the report will show the baseline issues, but since it was not caused by your PR they will not fail the report.

The primary goal is to maximize the signal to noise ratio and prevent the users from desiring to uninstall this workflow. It should be easy to adopt, and it should never nag the contributor about issues that their PR didn't create.

**TODO**: We will still figure out a way to expose those baseline errors yet.

## Cache Management

The tool incorporates caching initially intended to expedite repeated runs. This could be particularly useful for iterative development. Most runtime errors, especially those from the validate method and anchor checkers, can be debugged efficiently using cached data without re-fetching anything.

However, there's a known issue: the cache isn't always reliable. To ensure accuracy, always run `make clean-cache` between separate executions. The cache is still used to prevent repeated calls out and to avoid the repeated loading of HTML files into memory. As a result, a lot of memory can be used.

**TODO**: We're considering refining the cache management system. The cache should always be invalidated unless its validity is assured. This feature's primary purpose is for one-time use and might be phased out or redesigned in future versions.
