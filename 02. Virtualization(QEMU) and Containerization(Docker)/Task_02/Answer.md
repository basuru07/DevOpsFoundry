# Task 02 - Run Go HTTP Server in a Podman Scratch Container

# Scratch Container and Static vs Dynamic Applications

## Overview

A scratch container is a completely empty container image that contains no operating system, libraries, or utilities. It is primarily used for Go, C, or other statically compiled programs. Understanding the difference between static and dynamic applications is crucial for working with scratch containers.

## Dynamic vs Static Applications

### Dynamic Applications

Dynamic applications depend on shared libraries at runtime and require OS libraries to run. Since scratch containers have no OS libraries, dynamic binaries cannot execute in them. When you attempt to run a dynamic application in a scratch container, you will encounter an error:

```
bash: ./app: No such file or directory
```

**Solution:** Convert the dynamic application to a static binary before deploying it to a scratch container.

### Static Applications

Static applications have all dependencies compiled into the binary itself. The binary is self-contained and does not require any libraries from the container's operating system, making them ideal for scratch containers.

## Identifying Binary Type

To determine whether a binary is statically or dynamically linked, use the following command:

```bash
file app
```

The output will indicate either "statically linked" or "dynamically linked".

## Converting Dynamic to Static

To convert a dynamic Go binary to a static binary, use the following build command:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o app
```

Verify the conversion:

```bash
file app
# should output: "statically linked"
```

## Step-by-Step Task

### Step 1: Install Required Tools for Ubuntu

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install -y podman golang curl
```

### Step 2: Clone the Repository

```bash
git clone https://github.com/gnudeep/scratch-container.git
cd scratch-container
```

### Step 3: Understand the Project Structure

```bash
ls
```

### Step 4: Test the Application Locally

Before building, run the Go application locally to verify it works:

```bash
go run main.go
curl http://localhost:8080
```

Expected output: `Hello from Go app locally!`

### Step 5: Build the Go Application

First, check if the compiled binary is static or dynamic:

```bash
file app
```

If it is dynamically linked, convert it to static:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o app
```

Verify the binary is now statically linked:

```bash
file app
# should output: "statically linked"
```

### Step 6: Build the Container Image

```bash
podman build -t app-scratch-container .
```

### Step 7: Run the Container

Map the host machine port 6000 to the application running in the container on port 5000:

```bash
podman run -d -p 6000:5000 app-scratch-container
```

### Step 8: Check Container Status

Verify the container is running:

```bash
podman ps
```

### Step 9: Access the Application

Access the application from the host machine:

```bash
curl http://localhost:6000
```

Expected output: `Hello from Go app!`

### Step 10: Stop the Container

```bash
podman stop <container_id>
```

Replace `<container_id>` with the actual container ID from the `podman ps` output.