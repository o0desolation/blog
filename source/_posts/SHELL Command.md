## SHELL Command Collection

+ ssh

  ```
  ssh -l [username] [SERVER]
  ```

  ```
  ssh [username]@[SERVER]
  ```

+ scp 

  + download file

    ```
    scp [username]@[SERVER]:/[REMOTE_PATH]/[FILENAME] /[LOCAL_PATH]/
    ```

  + upload file

    ```
    scp /[LOCAL_PATH]/[FILENAME] [username]@[SERVER]:/[REMOTE_PATH]/
    ```

  + download dir

    ```
    scp -r [username]@[SERVER]:/[REMOTE_PATH]/ /[LOCAL_PATH]/
    ```

  + upload dir

    ```
    scp -r /[LOCAL_PATH]/ [username]@[SERVER]:/[REMOTE_PATH]/
    ```

+ Remote Script

  + bench.sh <!--I/O Info & Network SpeedTest-->

    ```
    wget -qO- bench.sh | bash
    curl -Lso- bench.sh | bash
    ```

    