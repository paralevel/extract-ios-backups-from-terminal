# Extract iOS backups with stock macOS CLI tools
Extracts backup files as APFS [clones](https://eclecticlight.co/2024/03/20/apfs-files-and-clones/) (which, unless they’re modified, don’t occupy any storage space), [deciphers](https://apple.stackexchange.com/questions/451511/how-to-view-iphone-backup-contents-without-a-3rd-party-app) the file names and places them into the original iOS subdirectory structure

<br>

1. Navigate to the backup folder (replace the last part with the subfolder‘s actual name)
~~~flf
cd $HOME/Library/Application\ Support/MobileSync/Backup/00000000-0000000000000000
~~~

- Get an overview of all the domains in the backup (a domain represents an specific location on the iOS filesystem, e.g. HomeDomain which equals /private/var/mobile/)
~~~flf
sqlite3 -csv Manifest.db "select domain from Files" | uniq | less 
~~~

3. Extract everything to `~/Desktop/ios-backup-extract`, excluding paths containing the strings AppDomain, CloudDocs and FileProvider, which should reduce the extraction time to somewhere around 10 minutes:
~~~flf
sqlite3 Manifest.db '.mode list' '.once /dev/stdout' 'select "find . -name " || fileID || " -print0 | xargs -0I{} ditto --clone {} ""'$HOME'/Desktop/ios-backup-extract/" || domain || "/" || relativePath || """" from files' | grep -v AppDomain | grep -v CloudDocs | grep -v FileProvider | sh
~~~

- Locate a specific cloned file’s corresponding original file 
> Use the last part of the path only, from the domain part, e.g. `HomeDomain/rest/of/path`, without the leading `/Users/username/Desktop/ios-backup-extract/`
~~~flf
sqlite3 Manifest.db '.once /dev/stdout' 'select fileID from files where concat(domain || "/" || relativePath) like "path/to/extracted/file"' 
~~~
~~~flf
find orig_bkp_dir -name matching_file_id -type f
~~~

