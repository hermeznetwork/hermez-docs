The main purpose of this file is to provide configuration (e.g. global parameters) used across the different state machines that compose the same project. It is typically called `config.pil` and it is equivalent to the global configuration that is typically used in other languages and stored in the source directory of user's projects.

For the shake of building the zkEVM, the configuration file is composed by:

```
constant %N = 2**21;
```

Here, $N$ is the upper bound on the number of rows that the distinct state machines that give form to the zkEVM are limited to.