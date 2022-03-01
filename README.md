## Github action for Contextual QA Checklists

Tests are nice, but sometimes you want an additional checklist of items to check before merging a PR
(for example, grammar check for documentation changes or last-minute check of visual look).
This action allows you to build filename-based checklists to remind the PR author about.

The action reads a checklist specification file (by default `CHECKLIST.yml`) from the repository root and submits a checklist comment to new/modified PRs based on what files were modified.

This is a modified version of https://github.com/wyozi/contextual-qa-checklist-action which supports multiple paths for a checklist. 

### Example

#### Checklist specification

`CHECKLIST.yml`

```yml
store:
  "paths":
    - "Store/**.js"
    - "Cart/**.html"
  "description":
    - Please check the following store pages (general)
  "items":
    - Store checklist 1
    - Store checklist 2
blog:
  "paths":
    - "Blog/**.php"
    - "Blogger/**.ts"
  "description":
    - Please check the following blog pages (general)
  "items":
    - Blog checklist 1
    - Blog checklist 2
```

#### Action

`.github/workflows/checklist.yml`

```yml
on: pull_request

jobs:
  checklist_job:
    permissions: 
      pull-requests: write
      contents: read
    runs-on: ubuntu-latest
    name: Checklist
    steps:
      - name: Checkout
        uses: actions/checkout@v1
      - name: Checklist  
        uses: batbattur/contextual-qa-checklist-action@master
        with:
          gh-token: ${{ secrets.GITHUB_TOKEN }}
          # For custom location of the checklist file. 
          input-file: '.github/QAchecklist.yml'
          # For custom comment header message
          comment-header: "Hey, by the way, we noticed the following changes:"
```

#### Options

##### `comment-header`

Overrides the default header text in the PR comment.

##### `comment-footer`

Overrides the default footer text in the PR comment.

##### `include-hidden-files`

Includes files and folders starting with `.` when matching. Defaults to `false`.

##### `input-file`

The path to the checklist definition file. Default to `CHECKLIST.yml` in the project root.

##### `gh-token`

The Github token for you project.

##### `show-paths`

Shows the matched file path in the PR comment. Defaults to `true`.

#### Result

When matching files are updated in a PR, the action will automatically post a checklist containing items under that path's key.

See https://github.com/batbattur/All/pull/36#issuecomment-1055868189 for an example PR checklist

#### File matching

File matcher uses https://github.com/isaacs/minimatch. Here is a cheat sheet for simple explanation https://github.com/motemen/minimatch-cheat-sheet

Example for the checklist file paths:
- `**.js` matches js file only in the root folder.
- `**/**.js` matches all js files in all subfolders.
- `Store/**.js` matches all js files in the `Store` folder.
- `Store/**/**.js` matches all js files in the `Store` folder and in its subfolders.

| Pattern | Matches | Does not match |
| ------- | ------- | -------------- |
| `**.js` | `Test.js` | `Subfolder1/Subolder2/Test.js` |
| `**/**.js` | `Test.js`, `Subfolder1/Subfolder2/Test.js` |  |
| `Store/**.js` | `Store/Test.js` | `Store/Subfolder/Test.js` |
| `Store/**/**.js` | `Store/Test.js`, `Store/Subfolder/Test.js` | |


