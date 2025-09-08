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

import requests, csv
from concurrent.futures import ThreadPoolExecutor

def get_repo_data(api_url, username, password):
    """
    Fetches the repository data from the Stash server.

    :param api_url: The API URL for the Stash repository
    :param username: The username for authentication
    :param password: The password for authentication
    :return: JSON data of the repository
    """
    response = requests.get(api_url, auth=(username, password))
    response.raise_for_status()  # Raise an error for bad status codes
    return response.json()

def find_pom_xml_in_repo(api_url, base_url, username, password):
    """
    Recursively search for pom.xml files in the repository data.

    :param api_url: The API URL for the current directory
    :param base_url: The base URL for the API to fetch file content
    :param username: The username for authentication
    :param password: The password for authentication
    :return: A list of paths to found pom.xml files
    """
    pom_files = []

    def recursive_search(url):
        while url:
            data = get_repo_data(url, username, password)
            for file_info in data.get('values', []):
                if isinstance(file_info, str):
                    if file_info.endswith('pom.xml'):
                        pom_files.append(file_info)
                elif isinstance(file_info, dict) and file_info['type'] == 'DIRECTORY':
                    subdir_url = f"{base_url}/{file_info['path']['toString']}"
                    recursive_search(subdir_url)
                elif isinstance(file_info, dict) and file_info['type'] == 'FILE':
                    if file_info['name'] == 'pom.xml':
                        pom_files.append(file_info['path']['toString'])

            # Handle pagination
            if data.get('isLastPage'):
                url = None
            else:
                next_page_start = data.get('nextPageStart')
                url = f"{api_url}?start={next_page_start}"

    recursive_search(api_url)
    return pom_files

def process_repository(repo_slug, project_key, base_api_url, username, password):
    """
    Process a single repository to find pom.xml files.

    :param repo_slug: The slug of the repository
    :param project_key: The project key of the repository
    :param base_api_url: The base API URL for the Stash server
    :param username: The username for authentication
    :param password: The password for authentication
    :return: A list of paths to found pom.xml files in the repository
    """
    stash_api_url = f"{base_api_url}/projects/{project_key}/repos/{repo_slug}/files"
    return find_pom_xml_in_repo(stash_api_url, stash_api_url, username, password)

def fetch_file_content(project_key, repo_slug, file_path, base_api_url, username, password):
    """
    Fetches the content of a file from the Stash server.

    :param file_path: The path of the file in the repository
    :param base_api_url: The base API URL for the Stash server
    :param username: The username for authentication
    :param password: The password for authentication
    :return: The content of the file as a string
    """
    file_url = f"{base_api_url}/projects/{project_key}/repos/{repo_slug}/browse/{file_path}?raw"
    response = requests.get(file_url, auth=(username, password))
    response.raise_for_status()
    return response.text

def find_version_tags_from_pom(pom_xml_content, version_tags):
    """
    Extracts the values of specified version tags from the pom.xml content.

    :param pom_xml_content: The content of the pom.xml file
    :param version_tags: List of version tag names to search for
    :return: A dictionary with version tag names as keys and their corresponding values, or None if not found
    """
    try:
        versions = {}

        for tag in version_tags:
            start_tag = f"<{tag}>"
            end_tag = f"</{tag}>"
            start_index = pom_xml_content.find(start_tag)
            end_index = pom_xml_content.find(end_tag)

            if start_index != -1 and end_index != -1:
                version_value = pom_xml_content[start_index + len(start_tag):end_index].strip()
                versions[tag] = version_value

        return versions if versions else None
    except Exception as e:
        print(f"Error parsing pom.xml: {e}")
        return None

def main():
    base_api_url = 'https://stash.cvent.net/rest/api/1.0'
    username = 'aradhya.batra'
    password = '***'
    project_keys_and_slugs = [
        ('EVT', 'attendee-order'),
    ('PT', 'oss-event-reporting-datasource'),
    ('PT', 'sms'),
    ('API', 'universal-appointments'),
    ('PT', 'public-api'),
    ('API', 'universal-gamification'),
    ('LS', 'ls-venue-compsets'),
    ('APPT', 'appointments-worker'),
    ('BT', 'bt-test-utils'),
    ('APPT', 'appointments'),
    ('LS', 'ls-reports-datasource'),
    ('CNF', 'conference-es-search'),
    ('LS', 'ls-adr'),
    ('CWITCH', 'chime-shared-library')
    ]
    csv_file = open('versions12.csv', 'w', newline='')  
    csv_writer = csv.writer(csv_file)  
    csv_writer.writerow(['Project Key', 'Repo Slug', 'File Path', 'Version Tag', 'Version'])

    version_tags = ['mono-java.version', 'postgresql.version', 'mssql.version', 'maven-parent']

    for project_key, repo_slug in project_keys_and_slugs:
        try:
            pom_files = process_repository(repo_slug, project_key, base_api_url, username, password)

            with ThreadPoolExecutor(max_workers=10) as executor:
                futures = [executor.submit(fetch_and_parse_pom, project_key, repo_slug, pom_file, base_api_url, username, password, version_tags) for pom_file in pom_files]

                for future in futures:
                    result = future.result()
                    if result:
                        pom_file, versions = result
                        if versions:
                            for tag, version in versions.items():
                                csv_writer.writerow([project_key, repo_slug, pom_file, tag, version])
            print()
        except Exception as e:
            print(f"Error processing {project_key}/{repo_slug}: {e}")

def fetch_and_parse_pom(project_key, repo_slug, pom_file, base_api_url, username, password, version_tags):
    try:
        pom_content = fetch_file_content(project_key, repo_slug, pom_file, base_api_url, username, password)
        versions = find_version_tags_from_pom(pom_content, version_tags)
        return (pom_file, versions) if versions else None
    except Exception as e:
        print(f"Error processing {project_key}/{repo_slug}/{pom_file}: {e}")
        return None

if __name__ == '__main__':
    main()
