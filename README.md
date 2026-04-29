# Extract iOS backup files with stock macOS CLI tools
Extract files from iOS backups using only stock command line tools on macOS (Tahoe 26)

<br>

1. Navigate to the backup folder (replace the last part with the subfolder‘s actual name)
~~~flf
cd $HOME/Library/Application\ Support/MobileSync/Backup/00000000-0000000000000000
~~~

- Get an overview of all the domains in the backup
~~~flf
sqlite3 -csv Manifest.db "select domain from Files" | uniq | less 
~~~

3. Extract everything as deobsfuscated [named](https://apple.stackexchange.com/questions/451511/how-to-view-iphone-backup-contents-without-a-3rd-party-app) APFS file [clones](https://eclecticlight.co/2024/03/20/apfs-files-and-clones/) (which, unless they’re modified, don’t occupy any storage space), to `~/Desktop/ios-backup-extract` (remember to delete this folder in advance everytime you run the command), excluding paths containing the strings AppDomain, CloudDocs and FileProvider, which should reduce the extraction time to somewhere around 10 minutes:
~~~flf
sqlite3 Manifest.db '.mode list' '.once /dev/stdout' 'select "find . -name " || fileID || " -print0 | xargs -0I{} ditto --clone {} ""'$HOME'/Desktop/ios-backup-extract/" || domain || "/" || relativePath || """" from files' | grep -v AppDomain | grep -v CloudDocs | grep -v FileProvider | sh
~~~

- Find the original backup file that corresponds to an extracted file
> Use the last part of the path only, from the domain part, e.g. `HomeDomain/rest/of/path`, without the leading `/Users/username/Desktop/ios-backup-extract/`
~~~flf
sqlite3 Manifest.db '.once /dev/stdout' 'select fileID from files where concat(domain || "/" || relativePath) like "path_to_extracted_file"' 
~~~
~~~flf
find orig_bkp_dir -name matching_file_id -type f
~~~
<br>
<br>
<br>
<br>
<br>
