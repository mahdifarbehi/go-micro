# Use a POSIX shell (macOS/Linux)
SHELL := /bin/sh

# Binaries (no .exe on Unix)
FRONT_END_BINARY ?= frontApp
BROKER_BINARY    ?= brokerApp

# Compose command (use docker compose if you're on v2)
DOCKER_COMPOSE ?= docker compose
# If you definitely use Compose v2, set: DOCKER_COMPOSE := docker compose

.PHONY: up up_build down build_broker build_front start stop logs_front

## up: starts all containers in the background without forcing build
up:
	@echo "Starting Docker services..."
	$(DOCKER_COMPOSE) up -d
	@echo "Docker services started!"

## up_build: stops compose (if running), builds all projects and starts compose
up_build: build_broker
	@echo "Stopping Docker services (if running)..."
	-$(DOCKER_COMPOSE) down
	@echo "Building (when required) and starting Docker services..."
	$(DOCKER_COMPOSE) up --build -d
	@echo "Docker images built and started!"

## down: stop docker compose
down:
	@echo "Stopping Docker services..."
	-$(DOCKER_COMPOSE) down
	@echo "Done!"

## build_broker: builds the broker binary as a linux executable (for container)
build_broker:
	@echo "Building broker binary (linux/amd64)..."
	cd ../broker-service && \
		GOOS=linux GOARCH=amd64 CGO_ENABLED=0 \
		go build -o $(BROKER_BINARY) ./cmd/api
	@echo "Done!"

## build_front: builds the front end binary for the host (mac/linux)
build_front:
	@echo "Building front end binary (host OS)..."
	cd ../front-end && \
		CGO_ENABLED=0 \
		go build -o $(FRONT_END_BINARY) ./cmd/web
	@echo "Done!"

## start: starts the front end (background) and writes a PID file
start: build_front
	@echo "Starting front end..."
	cd ../front-end && \
		nohup ./$(FRONT_END_BINARY) > front.log 2>&1 & echo $$! > front.pid
	@echo "Front end started (pid: $$(cat ../front-end/front.pid)). Logs: ../front-end/front.log"

## stop: stop the front end (uses PID file if present; falls back to pkill)
stop:
	@echo "Stopping front end..."
	@if [ -f ../front-end/front.pid ]; then \
		kill "$$(cat ../front-end/front.pid)" 2>/dev/null || true; \
		rm -f ../front-end/front.pid; \
	else \
		pkill -f "$(FRONT_END_BINARY)" 2>/dev/null || true; \
	fi
	@echo "Stopped!"

## logs_front: tail the front-end logs
logs_front:
	@echo "Tailing front end logs (Ctrl-C to stop)..."
	tail -f ../front-end/front.log
