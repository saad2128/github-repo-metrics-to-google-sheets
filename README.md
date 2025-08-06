# GitHub Repository Insights Automation

This n8n workflow automatically collects GitHub repository metrics and exports them to Google Sheets. It searches for repositories by programming language, gathers detailed statistics including code metrics, and organizes the data in a structured spreadsheet format.

## Features

### Repository Data Collection
- **Search Parameters**: Find repositories by programming language, sorted by stars, forks, or last updated
- **Pagination Support**: Process multiple pages of results with configurable limits
- **Rate Limiting**: Built-in delays to respect GitHub API limits

### Comprehensive Metrics
- **Basic Repository Info**: Name, description, URL, owner details
- **Engagement Metrics**: Stars, forks, watchers, open issues, open pull requests
- **Repository Properties**: Language, license, creation/update dates, size
- **Code Analysis** (optional):
  - Total lines of code
  - Non-empty lines count
  - Total characters
  - File count and analysis coverage
  - Average metrics per file

### Advanced Features
- **Accurate Issue/PR Counts**: Uses GitHub Search API to separate issues from pull requests (GitHub's default `open_issues_count` combines both)
- **Smart File Filtering**: Processes only relevant code files, skipping common directories like `node_modules`, `dist`, `.git`
- **Batch Processing**: Handles large repositories efficiently with file batching
- **Error Handling**: Continues processing even if individual files fail to fetch

## Prerequisites

### Required Credentials
1. **GitHub API Token**
   - Personal access token with `repo` scope for public repositories
   - Configure in n8n as "GitHub account" credential

2. **Google Service Account**
   - Service account with Google Sheets API access
   - Share your target spreadsheet with the service account email
   - Configure in n8n as "Google Service Account" credential

### Google Sheets Setup
1. Create a new Google Spreadsheet
2. Share it with your service account email (with Editor permissions)
3. Note the Spreadsheet ID from the URL
4. Create or identify the target sheet name

## Configuration

Update the **Configuration** node with your settings:

```javascript
{
  "programming_language": "javascript",    // Target language
  "sort_by": "stars",                     // Sort by: stars, forks, updated
  "spreadsheet_id": "YOUR_SHEET_ID",      // Google Sheets ID
  "sheet_name": "Sheet1",                 // Target sheet name
  "results_per_page": 30,                 // Results per page (max 100)
  "max_pages": 5,                         // Maximum pages to process
  "current_page": 1,                      // Starting page (keep as 1)
  "include_code_metrics": true,           // Calculate code metrics (slower)
  "max_file_size_kb": 500                 // Skip files larger than this
}
```

### Configuration Options

| Parameter | Description | Default | Notes |
|-----------|-------------|---------|-------|
| `programming_language` | Target programming language | `javascript` | Any GitHub-supported language |
| `sort_by` | Sort repositories by | `stars` | Options: `stars`, `forks`, `updated` |
| `spreadsheet_id` | Google Sheets document ID | - | Extract from spreadsheet URL |
| `sheet_name` | Target sheet name | `Sheet1` | Must exist in the spreadsheet |
| `results_per_page` | Results per page | `30` | Max 100 per GitHub API |
| `max_pages` | Maximum pages to process | `5` | Controls total repositories |
| `current_page` | Starting page number | `1` | Always start with 1 |
| `include_code_metrics` | Enable code analysis | `true` | Significantly slower when enabled |
| `max_file_size_kb` | File size limit | `500` | Skip larger files during analysis |

## Output Data Structure

The workflow exports the following columns to Google Sheets:

### Repository Information
- **Repository Name**: Short name of the repository
- **Full Name**: Owner/repository format
- **Owner**: Repository owner username
- **Description**: Repository description
- **URL**: GitHub repository URL

### Metrics
- **Stars**: Number of stars
- **Forks**: Number of forks
- **Open Issues**: Actual count of open issues (not including PRs)
- **Open Pull Requests**: Actual count of open pull requests
- **Watchers**: Number of watchers/subscribers
- **Language**: Primary programming language
- **Size (KB)**: Repository size in kilobytes

### Metadata
- **Created At**: Repository creation date
- **Updated At**: Last update timestamp
- **Last Push**: Last push timestamp
- **Default Branch**: Default branch name
- **Topics**: Repository topics (comma-separated)
- **License**: License name
- **Private**: Whether repository is private
- **Archived**: Whether repository is archived
- **Has Issues**: Whether issues are enabled
- **Has Wiki**: Whether wiki is enabled
- **Has Pages**: Whether GitHub Pages is enabled

### Code Metrics (when enabled)
- **Total Lines**: Total lines of code
- **Non-Empty Lines**: Lines with actual content
- **Total Characters**: Total character count
- **Code Files**: Number of code files found
- **Files Analyzed**: Successfully analyzed files
- **Failed Files**: Files that failed analysis
- **Attempted Files**: Total files attempted
- **Avg Lines/File**: Average lines per file
- **Avg Chars/File**: Average characters per file
- **Analysis Coverage %**: Percentage of files analyzed
- **Primary Language**: Detected primary language
- **Processed At**: Analysis timestamp

## Workflow Architecture

### Main Processing Flow
1. **Configuration**: Set search parameters and options
2. **Search GitHub**: Find repositories matching criteria
3. **Repository Details**: Fetch detailed information for each repo
4. **Issue/PR Separation**: Get accurate counts using Search API
5. **Code Metrics** (optional): Analyze repository contents
6. **Data Export**: Save to Google Sheets
7. **Pagination**: Continue to next page if available

### Code Metrics Pipeline
When enabled, the workflow:
1. Fetches repository file tree
2. Filters relevant code files
3. Processes files in batches (rate limiting)
4. Counts lines, characters, and other metrics
5. Aggregates results per repository

### Rate Limiting & Error Handling
- Built-in delays between API calls
- Continues on individual file failures
- Respects GitHub API rate limits
- Handles large repositories gracefully

## Usage Examples

### Basic Repository Search
Search for top JavaScript repositories:
```javascript
{
  "programming_language": "javascript",
  "sort_by": "stars",
  "results_per_page": 50,
  "max_pages": 2,
  "include_code_metrics": false
}
```

### Detailed Code Analysis
Analyze Python repositories with full metrics:
```javascript
{
  "programming_language": "python",
  "sort_by": "updated",
  "results_per_page": 10,
  "max_pages": 1,
  "include_code_metrics": true,
  "max_file_size_kb": 1000
}
```

### Large-Scale Collection
Collect extensive repository data:
```javascript
{
  "programming_language": "go",
  "sort_by": "forks",
  "results_per_page": 100,
  "max_pages": 10,
  "include_code_metrics": false
}
```

## Performance Considerations

### Execution Time
- **Without code metrics**: ~2-5 seconds per repository
- **With code metrics**: ~30-120 seconds per repository (depending on size)

### API Rate Limits
- GitHub API: 5,000 requests/hour for authenticated users
- The workflow includes automatic rate limiting
- Code metrics analysis uses additional API calls

### Recommendations
- Start with `include_code_metrics: false` for faster results
- Use smaller `results_per_page` when analyzing code metrics
- Monitor execution time and adjust `max_pages` accordingly

## Troubleshooting

### Common Issues

**Authentication Errors**
- Verify GitHub token has correct permissions
- Ensure Google Service Account has Sheets API access
- Check if spreadsheet is shared with service account

**Rate Limiting**
- Reduce `results_per_page` or `max_pages`
- Increase wait time in "Wait (Rate Limit)" node
- Monitor GitHub API rate limit headers

**Code Metrics Timeouts**
- Reduce `max_file_size_kb`
- Disable code metrics for large-scale collection
- Process fewer repositories per run

**Missing Data**
- Some repositories may have restricted access
- Private repositories require additional permissions
- Archived repositories may have limited data

### Debug Tips
- Check n8n execution logs for detailed error messages
- Verify spreadsheet permissions and sheet names
- Test with a small dataset first (`results_per_page: 1`)

## Customization

### Adding Custom Metrics
Modify the "Format Repo Data" node to include additional fields:
```javascript
custom_metric: repo.custom_field || 'default_value'
```

### Filtering Repositories
Add filters in the "Split Repos" node:
```javascript
// Example: Filter by minimum stars
if (repo.stargazers_count < 100) return null;
```

### Different Programming Languages
The workflow supports any GitHub-recognized language:
- `javascript`, `typescript`, `python`, `java`, `go`, `rust`
- `c`, `cpp`, `csharp`, `php`, `ruby`, `swift`, `kotlin`
- And many more

## License

This workflow is provided as-is for educational and research purposes. Please respect GitHub's Terms of Service and API rate limits when using this tool.
