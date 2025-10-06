# Producing a Docker image without a Dockerfile

## Key realisation

You can create a Docker image **without writing a Dockerfile** by working with a container. This is called **committing** &mdash; it saves the current state of a container as a new image, similar to saving changes in Git.

## The Lab

In this lab, we're asked to create a new image from `diamol/ch03-lab:2e`, after modifying the file at `/diamol/ch03.txt` to include our names.

## My Solution

1. **Run a container from the image**

Since the container is designed to perform a task and exit, we'll run it normally and let it finish:

```bash
docker container run --name lab3-container diamol/ch03-lab:2e
```

After this, the container will exit. But its filesystem is still there and accessible.

2. **Copy the file out of the exited container**

We want to update `/diamol/ch03.txt`, so we copy it to our local machine:

```bash
docker container cp lab3-container:/diamol/ch03.txt ./ch03.txt
```

3. **Edit the file locally**

Open `ch03.txt` in any editor and add your name(s), then save it and exit.

4. **Copy the updated file back into the container**

Now overwrite the file inside the exited container:

```bash
docker container cp ./ch03.txt lab3-container:/diamol/ch03.txt
```

5. **Commit the container as a new image**

Finally, create a new image that includes your changes:

```bash
docker container commit \
  -a "Mfon Kiwi Eti-mfon <mfonetimfon@gmail.com>" \
  -m "Created from 'diamol/ch03-lab:2e', but with the file 'ch03.txt' updated to include my name" \
  lab3-container \
  mfon/ch03-lab:custom
```

You now have a new image `mfon/ch03-lab:custom` with your updated file baked in.

### Did that work for real?

You may confirm this by running a container from the image with an interactive terminal:

```sh
docker container run -it mfon/ch03-lab:custom-v2 sh
```

Then, from inside the container, list the files and read the contents:

```sh
ls
```

```sh
cat ch03.txt
```
