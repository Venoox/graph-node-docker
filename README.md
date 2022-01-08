# Graph Node

Docker compose configuration includes:

- Graph Node (custom built)
- IPFS server
- Postgres Database
- Ethereum geth node

The purpose of this is to have a quick way to run your own Graph Node without configuring everything. It has all components needed to run the node but it will take a while to get the Ethereum node synced up. Alternatively you can remove the ethereum node from the docker compose and use a remote RPC endpoint.

Graph Node image includes environment variables for customization:

- POSTGRES_USER: username (default: `postgres`)
- POSTGRES_PASSWORD: password (default: `postgres`)
- POSTGRES_HOST: host (default: `localhost`)
- POSTGRES_PORT: port (default: `5432`)
- POSTGRES_DB: database created on startup (default: `graph-node`)

- IPFS_HOST: host (default: `localhost`)
- IPFS_PORT port (default: `5001`)

- RPC_URI_SCHEME: `http` or `https` (default: `http`)
- RPC_HOST: host (default: `localhost`)`
- RPC_PORT: port (default: `8545`)
- RPC_CAPABILITIES: can be empty or `full` or `archive` or any combination of these separated by commas
- RPC_NETWORK: network id or name (default: `mainnet`)
- RPC_URI: URI (example: http://localhost/node -> URI: `/node`)

# Development

Graph Node image is custom built with multi-stage build, other images are provided by their respective developer teams.
In the first stage, it uses a Rust image based on Debian Buster (slim) that contains the tools neeeded to build the binary application. It installs some needed dependencies, clones the repo and builds it.
After that in second stage, it uses a normal Debian Buster (slim) image and copies the binary from second stage and installs some runtime dependencies.

Issues I encountered:

- At first I was trying to run CMD in exec form a.k.a without shell because the application can receive Unix signals when shutting down the container gracefully. Oherwise the shell is considered PID 1 which prevents signals from reaching the app. But I needed environment variables when running the command and only the shell can do variable substitution so I switched to shell form even though it's not the best way.
- The main application (graph-node) was not made cloud native because it doesn't retry to connect to the Ethereum node but just fails to start and exists. I tried fixing this by using depends_on feature in the Docker Compose but that didn't work so I figured out depends_on only starts this container after it starts other containers and doesn't actually check if other containers are ready to accept connections so I needed another solution. While reading Docker docs I saw they recommend using a shell script that checks periodically if a TCP connection can be made and they even linked to some tools that already do that. I chose [wait-for-it](https://github.com/vishnubob/wait-for-it) mostly because it was first in the list and worked for me. I included the script in the repo and used it in the Dockerfile so that it checks for all other 3 components to be ready before starting it.
- Somewhat unrelated to Docker but the application couldn't connect to HTTPS endpoints because the slim image doesn't include CA certificates and the fix was to just install ca-certificates package provided by Debian.

This project is supposed to be used as a GraphQL server that processes requests from clients and blocks from the blockchain however it does include a website for GraphQL queries for easier access so that's only thing I can take a screenshot of.
