

# Creating Custom Sandbox

In this guide, we'll build a custom E2B sandbox with preinstalled dependencies
and files. Once the sandbox is built, we'll show how to spawn and control it
with our SDK.

##

Prerequisites

  1. Node.js 18.0.0 or later
  2. E2B CLI
  3. Docker

##

1\. Install E2B CLI

Run

    
    
    npm install -g @e2b/cli@latest
    

CopyCopied!

You need Node.js 18.0.0 or later to install the CLI.

## 2\. CLI login

Before you create your first custom sandbox, you will need to authenticate in
the CLI with your E2B account. Run the following command in your terminal.

Run

    
    
    e2b login
    

CopyCopied!

## 3\. Create sandbox template file

To describe how your custom sandbox will look like, you create a new
Dockerfile and name it `e2b.Dockerfile`. We use this Dockerfile as the sandbox
template file.

Run `e2b init` to create a basic `e2b.Dockerfile` in the current directory.

We want to have ffmpeg installed in our custom sandbox, so we'll create the
`e2b.Dockerfile` with the following content.

Run

    
    
    # You can use most of the Debian-based base images
    FROM ubuntu:22.04
    
    # Install dependencies and customize sandbox
    RUN apt update \
    	&& apt install -y sudo \
      && sudo apt install -y ffmpeg
    

CopyCopied!

## 4\. Build custom sandbox

Now it's time to create your custom sandbox based on the sandbox template file
(the `e2b.Dockefile` file) you just created in the previous step.

Run the following command inside the template file directory in your terminal.

Run

    
    
    e2b build
    

CopyCopied!

Use the `.dockerignore` file to exclude files from the sandbox template.

The final output should look similar to this.

Run

    
    
    Preparing sandbox template building (113 files in Docker build context).
    Found ./e2b.Dockerfile that will be used to build the sandbox template.
    Started building the sandbox template 3y5bvra6kgq0kaumgztu
    
    Step 1/4 : FROM ubuntu:22.04
     ---> e4c58958181a
    Step 2/4 : RUN apt update 	&& apt install sudo
     ---> Using cache
     ---> 4762cc0db1ff
    Step 3/4 : RUN type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
     ---> Using cache
     ---> 384706ddfe2f
    Step 4/4 : RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg 	&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg 	   && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null 	 sudo apt update 	 sudo apt install gh -y
     ---> Using cache
     ---> b8649b84c829
    Successfully built b8649b84c829
    Successfully tagged us-central1-docker.pkg.dev/e2b-prod/custom-environments/3y5bvra6kgq0kaumgztu:b96935dc-fe87-45fb-229d-a3fba4fe160d
    
    Running postprocessing. It can take up to few minutes.
    
    ✅ Building sandbox template 3y5bvra6kgq0kaumgztu finished.
    

CopyCopied!

This will create the `e2b.toml` file storing the sandbox config.

Run

    
    
    # This is a config for E2B sandbox template
    id = "3y5bvra6kgq0kaumgztu"
    dockerfile = "e2b.Dockerfile"
    

CopyCopied!

**Our custom sandbox ID is`3y5bvra6kgq0kaumgztu` in this case.**

### Updating your sandbox

If you want to update your sandbox, you run the same command you did to build
it. This will rebuild the image and update your sandbox template. You don't
need to specify the `--dockerfile` flag again.

Run

    
    
    e2b build
    

CopyCopied!

## 5\. Spawn and control your sandbox

Now you can use the E2B SDK to control your new custom sandbox.

Our custom sandbox ID is `3y5bvra6kgq0kaumgztu` from the previous step. We'll
just pass it to the SDK.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    import { Sandbox } from '@e2b/sdk'
    
    const sandboxID = '3y5bvra6kgq0kaumgztu'
    
    const sandbox = await Sandbox.create({
      id: sandboxID,
    })
    
    await sandbox.close()
    

CopyCopied!

Send feedback to CEO

© FoundryLabs, Inc. 2023. All rights reserved.

548 Market Street #83122, San Francisco, CA 94104

Follow us on TwitterFollow us on GitHubJoin our Discord server

