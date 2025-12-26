SERVER: filesystem-test
PORT: Native MCP (stdio only)
PURPOSE: Native Anthropic MCP file operations for /Users/macbook/Documents

TOOL: read_file
INPUT: { path: string (required) }
OUTPUT: { contents: string }
USE: Read file contents at specified path
EXAMPLE: read_file({ path: "/Users/macbook/Documents/file.md" })

TOOL: read_multiple_files
INPUT: { paths: string[] (required) }
OUTPUT: { results: [{ path, contents?, error? }] }
USE: Read multiple files in parallel
EXAMPLE: read_multiple_files({ paths: ["/path/a.md", "/path/b.md"] })

TOOL: write_file
INPUT: { path: string (required), content: string (required) }
OUTPUT: { success: boolean, path }
USE: Write content to file (creates or overwrites)
EXAMPLE: write_file({ path: "/path/new.md", content: "# New Document" })

TOOL: edit_file
INPUT: { path: string (required), edits: [{ old_text: string, new_text: string }] (required), dry_run?: boolean }
OUTPUT: { success: boolean, changes_made: number }
USE: Edit file with find/replace operations
EXAMPLE: edit_file({ path: "/path/file.md", edits: [{ old_text: "foo", new_text: "bar" }] })

TOOL: create_directory
INPUT: { path: string (required) }
OUTPUT: { success: boolean, path }
USE: Create a new directory
EXAMPLE: create_directory({ path: "/path/new-folder" })

TOOL: list_directory
INPUT: { path: string (required) }
OUTPUT: { entries: [{ name, type: "file"|"directory" }] }
USE: List contents of a directory
EXAMPLE: list_directory({ path: "/Users/macbook/Documents" })

TOOL: directory_tree
INPUT: { path: string (required), depth?: number }
OUTPUT: { tree: string }
USE: Get directory tree structure
EXAMPLE: directory_tree({ path: "/Documents/project", depth: 3 })

TOOL: move_file
INPUT: { source: string (required), destination: string (required) }
OUTPUT: { success: boolean, from, to }
USE: Move or rename a file
EXAMPLE: move_file({ source: "/path/old.md", destination: "/path/new.md" })

TOOL: search_files
INPUT: { path: string (required), pattern: string (required), exclude_patterns?: string[] }
OUTPUT: { matches: [{ path, line?, content? }] }
USE: Search for files matching pattern
EXAMPLE: search_files({ path: "/Documents", pattern: "*.md" })

TOOL: get_file_info
INPUT: { path: string (required) }
OUTPUT: { path, size, created, modified, type }
USE: Get file metadata
EXAMPLE: get_file_info({ path: "/path/file.md" })

SCOPE: /Users/macbook/Documents only
TYPE: Native Anthropic MCP server (official implementation)
PERMISSIONS: Read, write, create, move within scope
