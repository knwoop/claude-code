# Claude Code Global Guidelines

## Plan Mode Output Format

When creating a plan in plan mode, use the following format:

### 1. Requirements
  Purpose and overview of the feature to implement
  List specifications such as statuses or state transitions as needed

### 2. TODO
Group tasks by package/module:

#### package_name_1
  [ ] Task name
      → Target file and implementation approach
      → Include logic or code examples as needed:
         ```go
         func Example() error {
             // implementation example
         }
         ```

#### package_name_2
  [ ] Task name
      → Implementation approach (can be multiple lines)
      → Details of conditions or logic:
         1. If condition A → Process A
         2. If condition B → Process B

### 3. References
List related resources including files, PRs, and documents:

#### Files
  `path/to/file.go` → Reason
  `path/to/another.go` → Reference for similar implementation

#### PRs / Commits
  https://github.com/org/repo/pull/123 → PR for similar feature
  https://github.com/org/repo/commit/abc123 → Reference implementation

#### Documents
  https://notion.so/xxx → Design Doc
  https://notion.so/yyy → PRD

### 4. Verification
Include decision table (when applicable) and test commands:

#### Decision Table (for complex conditional logic)
| Condition A | Condition B | Condition C | Expected Result |
|-------------|-------------|-------------|-----------------|
| Y           | Y           | -           | Result 1        |
| Y           | N           | Y           | Result 2        |
| N           | -           | -           | Result 3        |

#### Test Commands
  `go test -run TestXxx ./...`
