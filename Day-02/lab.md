# Lab &mdash; Replace the Homepage Content

## The Task

We were asked to run the website container from the chapter, but replace the `index.html` file so that when we browse to the container, we see a different home page.

## Hindsight

I wrongly convinced myself that one could only copy files _out_ of a container &mdash; not _into_ one &mdash; so I ended up editing the file directly from inside the container.

Also, for some reason I missed the keyword "REPLACE" in the assignment, and so editing the file seemed appropriate.

Now I know there are other ways to do this, but here's what I did.

## My Solution

1. Run the container in detached mode, publishing host port `8088` to container port `80` &mdash; and take note of the container ID:

   ```sh
   docker container run -dp 8088:80 diamol/ch02-hello-diamol-web:2e
   ```

2. Open an interactive shell inside the running container using the container ID:

   ```sh
   docker container exec -it <container-id> sh
   ```

3. From the interactive shell, explore the filesystem using commands like `ls`, `cd`, and `cat` until you locate the HTML file being served.

4. Open the `index.html` file with `vi`:

   ```sh
   vi htdocs/index.html
   ```

5. Edit and save the file:

   - Press `i` to enter insert mode
   - Make your edits
   - Press `ESC`, type `:wq`, and hit `ENTER` (this Writes your changes to the file and Quits the editor)

6. Visit `http://localhost:8088` in your browser and admire your new home page

---

This method worked &mdash; and it gave me a reason to (re)learn a bit of `vi`, so no regrets.
