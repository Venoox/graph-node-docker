FROM rust:1.57.0-slim-buster AS builder

RUN apt-get update && apt-get -y --no-install-recommends install git clang libpq-dev libssl-dev pkg-config
RUN rustup component add rustfmt
RUN git clone https://github.com/graphprotocol/graph-node.git
WORKDIR /graph-node
RUN ["cargo", "build", "-p", "graph-node", "--release"]

FROM debian:buster-20211220-slim
RUN apt-get update && apt-get install -y libpq5 ca-certificates
COPY --from=builder /graph-node/target/release/graph-node ./

ENV POSTGRES_USER postgres
ENV POSTGRES_PASSWORD postgres
ENV POSTGRES_HOST localhost
ENV POSTGRES_PORT 5432
ENV POSTGRES_DB graph-node

ENV IPFS_HOST localhost
ENV IPFS_PORT 5001

ENV RPC_URI_SCHEME http
ENV RPC_HOST localhost
ENV RPC_PORT 8545
ENV RPC_CAPABILITIES ""
ENV RPC_NETWORK mainnet
ENV RPC_URI ""

ADD ./wait-for-it.sh ./
RUN chmod +x ./wait-for-it.sh

CMD ./wait-for-it.sh ${IPFS_HOST}:${IPFS_PORT} && \
    ./wait-for-it.sh ${POSTGRES_HOST}:${POSTGRES_PORT} && \
    ./wait-for-it.sh ${RPC_HOST}:${RPC_PORT} && \
    exec ./graph-node --postgres-url postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB} \
    --ethereum-rpc ${RPC_NETWORK}:${RPC_CAPABILITIES}:${RPC_URI_SCHEME}://${RPC_HOST}:${RPC_PORT}${RPC_URI} \
    --ipfs ${IPFS_HOST}:${IPFS_PORT}