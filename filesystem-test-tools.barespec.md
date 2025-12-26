TOOLS: filesystem-test
VERSION: 1.1
UPDATED: 2025-12-20
COUNT: 12

---

TOOL: read_file
INPUT: { path: string (required) }
OUTPUT: { contents: string }
USE: Read file contents at specified path
EXAMPLE: read_file({ path: "/Users/macbook/Documents/file.md" })
NOTES: Returns full file contents as string.

---

TOOL: read_multiple_files
INPUT: { paths: string[] (required) }
OUTPUT: { results: [{ path: string, contents?: string, error?: string }] }
USE: Read multiple files in parallel
EXAMPLE: read_multiple_files({ paths: ["/path/a.md", "/path/b.md"] })
NOTES: Continues reading other files if one fails.

---

TOOL: write_file
INPUT: { path: string (required), content: string (required) }
OUTPUT: { success: boolean, path: string }
USE: Write content to file (creates or overwrites)
EXAMPLE: write_file({ path: "/path/new.md", content: "# New Document" })
NOTES: Creates parent directories if needed.

---

TOOL: edit_file
INPUT: { path: string (required), edits: [{ old_text: string, new_text: string }] (required), dry_run?: boolean }
OUTPUT: { success: boolean, changes_made: number }
USE: Edit file with find/replace operations
EXAMPLE: edit_file({ path: "/path/file.md", edits: [{ old_text: "foo", new_text: "bar" }] })
NOTES: All edits must match exactly. dry_run shows what would change.

---

TOOL: create_directory
INPUT: { path: string (required) }
OUTPUT: { success: boolean, path: string }
USE: Create a new directory
EXAMPLE: create_directory({ path: "/path/new-folder" })
NOTES: Creates parent directories if needed (recursive).

---

TOOL: list_directory
INPUT: { path: string (required) }
OUTPUT: { entries: [{ name: string, type: "file"|"directory" }] }
USE: List contents of a directory
EXAMPLE: list_directory({ path: "/Users/macbook/Documents" })
NOTES: Does not recurse into subdirectories.

---

TOOL: directory_tree
INPUT: { path: string (required), depth?: number }
OUTPUT: { tree: string }
USE: Get directory tree structure
EXAMPLE: directory_tree({ path: "/Documents/project", depth: 3 })
NOTES: Returns ASCII tree representation.

---

TOOL: move_file
INPUT: { source: string (required), destination: string (required) }
OUTPUT: { success: boolean, from: string, to: string }
USE: Move or rename a file
EXAMPLE: move_file({ source: "/path/old.md", destination: "/path/new.md" })
NOTES: Works for both move and rename operations.

---

TOOL: search_files
INPUT: { path: string (required), pattern: string (required), exclude_patterns?: string[] }
OUTPUT: { matches: [{ path: string, line?: number, content?: string }] }
USE: Search for files matching pattern
EXAMPLE: search_files({ path: "/Documents", pattern: "*.md" })
NOTES: Pattern is glob-style. Returns matching file paths.

---

TOOL: get_file_info
INPUT: { path: string (required) }
OUTPUT: { path: string, size: number, created: timestamp, modified: timestamp, type: string }
USE: Get file metadata
EXAMPLE: get_file_info({ path: "/path/file.md" })

---

TOOL: read_media_file
INPUT: { path: string (required) }
OUTPUT: { data: string (base64), mimeType: string, size: number }
USE: Read binary media file (images, audio)
EXAMPLE: read_media_file({ path: "/Documents/image.png" })
NOTES: Returns base64-encoded data with detected MIME type.

---

TOOL: list_directory_with_sizes
INPUT: { path: string (required), sort_by?: "name"|"size" (default: "name"), sort_order?: "asc"|"desc" (default: "asc") }
OUTPUT: { entries: [{ name: string, type: "file"|"directory", size?: number, size_human?: string }], total_size: number }
USE: List directory with file sizes
EXAMPLE: list_directory_with_sizes({ path: "/Documents/project", sort_by: "size", sort_order: "desc" })
NOTES: Directories show cumulative size. size_human is formatted (e.g., "1.2 MB").

---

SCOPE: /Users/macbook/Documents only
TYPE: Native Anthropic MCP server
PERMISSIONS: Full read/write within scope
