# Copilot Instructions for ContosoMCP

This is a .NET 8 MCP (Model Context Protocol) server project.

## Project Structure

- `ContosoMCP/` - Main project folder
  - `Program.cs` - Entry point and MCP server configuration
  - `Tools/` - MCP tool implementations
  - `ContosoMCP.csproj` - Project file

## Conventions

- Use C# 12 features and .NET 8 APIs
- Follow MCP SDK patterns for tool definitions
- Tools should be placed in the `Tools/` folder
- Use `[McpServerToolType]` and `[McpServerTool]` attributes for tool registration

## Building

```bash
dotnet build ContosoMCP/ContosoMCP.csproj
```

## Publishing

This project is published to NuGet.org as a dotnet tool.
