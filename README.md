# Docker

Issues I encountered:

- ENV doesn't work if you use CMD with array of strings because it's not run in a shell therefore there is not variable substitution
- The main software (graph-node) was not made cloud natively because it doesn't retry to connect to the database/eth node/ipfs but just fails to start
and even if I set depends_on in Compose that only waits for other containers to start but not actually be able to connect to and that's why I took a wait-for-it script from github repo (https://github.com/vishnubob/wait-for-it) that waits for TCP connection before continuing