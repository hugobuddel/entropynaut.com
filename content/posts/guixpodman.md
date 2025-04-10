+++
date = '2025-01-10T23:55:15+01:00'
draft = false
title = 'Podman on Guix'
+++

## Podman on Guix

Today was the day to switch from Docker to Podman!

This is the first article in the sharpening-the-saw series. Personally stories about keeping your tools up-to-date.

The trigger to move away from docker was fairly trivial: I [couldn't create a volume from `.`](https://github.com/tcort/markdown-link-check/pull/440) using docker 20.10. It was possible to use `$(pwd)` instead of a period, but it was the thought that mattered. Then it became clear that [docker engine 20.10 is end-of-life since December 2023](https://endoflife.date/docker-engine)...

I already tried some half hearted attempts of updating docker in Guix, but that turned out too difficult, due to circular dependencies if I recall correctly. In fact, it was pretty difficult to keep docker and docker-compose running at all on Guix, [my `config.scm` was full with inferiors](https://unix.stackexchange.com/q/698811/160100).

Furthermorer, podman supposedly had a rootless mode now, which I was excited to try, it didn't feel right to have these privilidged containers on my pristine Guix environment!

It took me a little while to find the right documentation. The internet pointed me to an [excellent blog by Josep Bigorra](https://jointhefreeworld.org/blog/articles/gnu-linux/podman-root-less-guix/index.html), which made me confident the switch is possible.
But that solition wasn't Guixy enough for me, writing files instead of having the configuration directly in scheme.

Luckily, I'm doing this exercise at exactly the right time, because [improved podman support landed in Guix just last month](https://guix.gnu.org/manual/devel/en/html_node/Miscellaneous-Services.html#index-Rootless-Podman). And it is pretty Guixy, essentially just this:
```guile
           (service rootless-podman-service-type
                    (rootless-podman-configuration
                     (subgids
                      (list (subid-range (name "hugo"))))
                     (subuids
                      (list (subid-range (name "hugo"))))))
```

So now I'm finally running all my containers through podman!

One surprising thing is that all the mounted volumes are owned by root. But that is probably fine, because being root in the container is perfectly harmless, since that doesn't mean root access on the host anymore!

I use containers for development work, mounting volumes from the host with code repositories and such. This required the user in the container to be the same as the user outside the container, making reusing the same ~Dockerfile~ Containerfile non-trivial.

Instead, I'll experiment with running everything in the containers as 'root' for a while.
