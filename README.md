# Docker images

A set of Docker images to use in the development of my personal projects. All available at [hub.docker.com](hub.docker.com). **Always work in progress**.

## Usage example

Let's pretend that you're developing a `Phoenix` application and you want to create a basic `Dockerfile` for development purposes. The code of you `Dockerfile` will look like this:

```
FROM dreamingechoes/elixir:1.5.1

# Install hex, rebar, and get deps
ENV MIX_ENV=dev

RUN mix local.hex --force && \
    mix local.rebar --force && \
    mix hex.info

WORKDIR /APP_PATH
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/dreamingechoes/docker-images. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The repository is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
