#  jupyterhub

## Networking
- configurable-http-proxy

## Authentication
- oauthenticator - GitHub OAuth and JupyterHub Authenticator
- ldapauthenticator
- [Custom authenticators in JupyterHub Wiki](https://github.com/jupyterhub/jupyterhub/wiki/Authenticators)
    - OAuth
         - GitHub/BitBucket/MediaWiki/CILogon OAuth
         - Google OAuth
         - Local+Google OAuth (from within a Docker container)
    - Other
         - Dummy Authenticator (For testing only! Not for production use.)
         - LDAP Authenticator
         - REMOTE_USER Authenticator (For when intermediate login infrastructure such as Apache offloads authentication and forwards REMOTE_USER header.)

## Spawners
- sudospawner
- dockerspawner
- Custom Spawners for JupyterHub
    - BatchSpawner for spawning remote servers using batch systems (Torque, PBS, Slurm, etc)
    - DockerSpawner, which actually has two different spawners in it:
         - dockerspawner.DockerSpawner, for spawning identical Docker containers for each user
         - dockerspawner.SystemUserSpawner, for spawning Docker containers with an environment and home directory for each user
    - RemoteSpawner
    - SimpleSpawner, for testing purposes.
    - SudoSpawner
    - SwarmSpawner, for spawning containers using Docker Swarm

## Tutorials and Reference Deployments
- jupyterhub-tutorial
- jupyterhub-deploy-docker
- jupyterhub-deploy-teaching

- jupyterhub-carina - Rackspace JupyterHub integration