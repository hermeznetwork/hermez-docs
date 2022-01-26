```yaml
version: "3.3"
services:
  hermez-db-test:
    image: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: "hermez"
      POSTGRES_PASSWORD: "yourpasswordhere"
  privatebc:
    image:  public.ecr.aws/r7d5k1t8/hermez-geth:latest
    ports:
      - "8545:8545"
    environment:
      - DEV_PERIOD
    entrypoint: ["geth", "--http", "--http.addr", "0.0.0.0","--http.corsdomain",
      "*", "--http.vhosts" ,"*", "--ws", "--ws.origins", "*", "--ws.addr", "0.0.0.0",
      "--dev", "--datadir", "/geth_data$DEV_PERIOD"]
```

