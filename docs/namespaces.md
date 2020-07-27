# Working with namespaces in Linux

- Usefull command to list all the current namespaces mounts in a tree view:
    - `findmnt`

## Mount namespace
How to create a mount namespace:
    - Create a folder that you want to bind to: 
        - `mkdir mounted`
    - Run command to unshare with **mount** flag: 
        - `sudo unshare --mount`
    - Run command to bind a folder to the **mounted** folder created previously: 
        - `mount --bind <path-to-folder> /mounted`
    - Create a file in the mounted folder
    - Now exit the unshared process and go to **/mounted** folder. It will be empty
    - If you modify an existing file or delete it, it will be deleted in the "real world"