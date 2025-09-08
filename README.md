"""
MAVEN DEPENDENCY VERSION SCANNER
=================================

This script is an automated tool for scanning multiple Git repositories in Atlassian Stash/Bitbucket
to extract and analyze Maven dependency versions from pom.xml files across different projects.

MAIN FUNCTIONALITY:
------------------
1. **Multi-Repository Scanning**: Automatically scans 14 different repositories across various projects:
   - EVT (Event Management), PT (Platform Tools), API (Universal APIs)
   - LS (Location Services), APPT (Appointments), BT (Build Tools)
   - CNF (Conference), CWITCH (Chime Integration)

2. **Recursive pom.xml Discovery**: 
   - Traverses entire repository directory structures recursively
   - Handles pagination for large repositories automatically
   - Finds all pom.xml files regardless of nested depth

3. **Targeted Version Extraction**: Searches for specific Maven dependency versions:
   - mono-java.version: Internal Java framework version
   - postgresql.version: PostgreSQL database driver version
   - mssql.version: Microsoft SQL Server driver version  
   - maven-parent: Parent POM version for dependency management

4. **Concurrent Processing**: 
   - Uses ThreadPoolExecutor with 10 worker threads for parallel processing
   - Significantly reduces scanning time across multiple repositories
   - Handles network timeouts and API rate limiting gracefully

5. **CSV Report Generation**:
   - Exports findings to 'versions12.csv' with structured data
   - Columns: Project Key, Repository Slug, File Path, Version Tag, Version Value
   - Easy integration with spreadsheet tools for analysis

6. **Enterprise Integration**:
   - Authenticates with Atlassian Stash REST API
   - Handles enterprise security and access controls
   - Raw file content fetching for accurate parsing

USE CASES:
----------
- **Dependency Auditing**: Track versions across microservices for security updates
- **Compliance Reporting**: Generate reports for license and vulnerability management  
- **Migration Planning**: Identify repositories using outdated dependencies
- **Standardization**: Ensure consistent versions across related projects

OUTPUT EXAMPLE:
--------------
Project Key | Repo Slug        | File Path      | Version Tag        | Version
EVT         | attendee-order   | pom.xml        | mono-java.version  | 2.1.5
PT          | public-api       | core/pom.xml   | postgresql.version | 42.3.1
...

The script provides comprehensive visibility into Maven dependency landscapes across
enterprise Git repositories for better dependency governance and security management.
"""
