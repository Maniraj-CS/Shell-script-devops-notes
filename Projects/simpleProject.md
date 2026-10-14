# Project 1

The project is about , let's create a folder if folder is not exist then create it otherwise say The code is exist due to folder is already existed.

```bash
#!/bin/bash

createDir(){
    mkdir demo
}


if ! createDir; then
    echo "The code is exist due to folder is already existed...."
    exist 1
fi

echo "Folder created successfully"
```

