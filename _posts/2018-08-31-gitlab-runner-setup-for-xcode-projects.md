---
title: "Setup Gitlab Runner for an Xcode Project"
date: 2018-08-31 07:24:00 -0800
categories: continious-integration gitlab mac xcode gitlab 
---

Setting up a gitab runner on a mac is fairly straightforward and the [official documentation](https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/install/osx.md) is actually quite good.
It does involve bouncing around a few places and looking someome things up.  
After setting up the nth runner myself I wanted to list all the steps I took in one place, and if I'm being honest I really should write a script automate this process if I keep doing it.  

### Download

Currently there is no easy one liner like ```brew install gitab-runner``` we can run to get this up and running.
But it's easy to do what homebrew might do for us, we can download the latest gitab-runner binary to ```/usr/local/bin``` using curl:

```sh
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
```

Then we need to allow it to run:

```sh
sudo chmod +x /usr/local/bin/gitlab-runner
```

### Register

Now we can register our runner with our gitlab server.  
Before we do that we'll need the url of our server a token which can be found 
in our gitlab project main menu |> Settings |> CI/CD |> Runner Settings.   
There is a box labelled "Setup a specific Runner manually" which has all the goods.

Okay so let's register:

```sh
gitlab-runner register
```

The first two items we just found in our runner settings, coordinator URL and gitlab runner CI token.   
Then give it a meaning give it a meaningful name and some tags.

Little bit about tags...tags are how gitlab will choose which runner to run for each of our CI jobs.  
Each job will have tags and each runner will have tags.  
When CI is triggered it runs down the list of jobs, if the job is able it run, it looks for all runners that have tags that match the tags for the job, which it finds one it starts the job on that runner.
So if we have a job call ```unit_tests``` and it has a tag ```ios_ci``` if we want this new runner to run that job should add the tag ```ios_ci```

Last step is choosing how the runner will execute.  Currently the only way I know of to run CI is through executing ```xcode-build``` via the shell so we'll choose ```shell``` from the list of options.

Now that its all registered we can start it up and it's good to go:

```sh
gitlab-runner start
```

We can verify this by going back to our CI settings in our gitlab project.  
If we expand runners there we should see our new runner listed with a green light icon next to it.

