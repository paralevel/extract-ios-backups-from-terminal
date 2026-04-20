# Extract iOS backup files with stock macOS CLI tools
Extract iOS backup files using only stock command line tools on macOS Tahoe 26

<br>

1. Navigate to the backup folder
~~~flf
cd $HOME/Library/Application\ Support/MobileSync/Backup/<uid>
~~~

- Get an overview of all the domains in the backup
~~~flf
sqlite3 -csv Manifest.db "select domain from Files" | uniq | less 
~~~


2. Extract everything except fileprovider and appdomain files to `~/Desktop/ios-bkp-extract.<uid>` – as APFS file [clones](https://eclecticlight.co/2024/03/20/apfs-files-and-clones/) (which don't occupy any storage space) with deobsfuscated [names](https://apple.stackexchange.com/questions/451511/how-to-view-iphone-backup-contents-without-a-3rd-party-app) – may take somewhere around 10 minutes
~~~flf
sqlite3 Manifest.db '.mode list' '.once /dev/stdout' 'select "find . -name " || fileID || " -print0 | xargs -0I{} ditto --clone {} ""'$(mktemp -d -t ios-bkp-extract -p $HOME/Desktop)'/" || domain || "/" || relativePath || """" from files' | grep -v "File Provider Storage" | grep -v AppDomain | sh
~~~

- Find the original backup file that corresponds to an extracted file
> Use the last part of the path only, from the domain part, e.g. `HomeDomain/rest/of/path`, without the leading `/Users/username/Desktop/ios-bkp-extract.<uid>/`
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
