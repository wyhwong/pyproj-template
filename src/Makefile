export DOCKER_BUILDKIT=1

# For Docker image
export VERSION?=0.1.0
export USERNAME?=$(shell whoami)
export USER_ID?=$(shell id -u)
export GROUP_ID?=$(shell id -g)
export SRC_DIR?=app
export DOCKERFILE?=./deployment/Dockerfile

# Environment variables
export TZ?=UTC
export STREAMING_LOG_LEVEL?=30
export FILE_LOG_LEVEL?=10
export LOG_FILEPATH?=./runtime.log

# Static code analysis / test
export TEST_REPORT_DIR?=./test_report

build:
	@echo "Building Docker image (MAIN)..."
	docker build \
		--build-arg VERSION=${VERSION} \
		--build-arg USERNAME=${USERNAME} \
		--build-arg USER_ID=${USER_ID} \
		--build-arg GROUP_ID=${GROUP_ID} \
		-t ${USERNAME}/${SRC_DIR}:${VERSION} \
		-f ${DOCKERFILE} \
		--no-cache \
		--target main .
	@echo "Building Docker image (TEST)..."
	docker build \
		--build-arg VERSION=${VERSION} \
		--build-arg USERNAME=${USERNAME} \
		--build-arg USER_ID=${USER_ID} \
		--build-arg GROUP_ID=${GROUP_ID} \
		-t ${USERNAME}/${SRC_DIR}-test:${VERSION} \
		-f ${DOCKERFILE} \
		--target test .

run:
	@echo "Running Docker container..."
	docker run --rm -it -d --name ${USERNAME}-${SRC_DIR} \
		-e TZ=${TZ} \
		-e STREAMING_LOG_LEVEL=${STREAMING_LOG_LEVEL} \
		-e FILE_LOG_LEVEL=${FILE_LOG_LEVEL} \
		-e LOG_FILEPATH=${LOG_FILEPATH} \
		${USERNAME}/${SRC_DIR}:${VERSION}

clean:
	@echo "Removing Docker container..."
	docker stop ${USERNAME}-${SRC_DIR}

# Install Python dependencies in poetry environment
install:
	@echo "Installing Python dependencies..."
	poetry install
	poetry run pre-commit install -f

# Upate Python dependencies in poetry environment
update:
	@echo "Updating Python dependencies..."
	poetry update

# Format analyticks using black, isort
format:
	@echo "Formatting source code..."
	poetry run black ${SRC_DIR}
	poetry run isort ${SRC_DIR}
	poetry run black tests
	poetry run isort tests

test:
	@echo "Running tests..."
	poetry run pytest \
		--disable-warnings \
		--cov-report=html:${TEST_REPORT_DIR} \
		--cov=${SRC_DIR} .

# Run static code analysis
static-code-analysis:
	@echo "Running static code analysis..."
	mkdir -p ${TEST_REPORT_DIR}
	poetry install
	poetry run black ${SRC_DIR} --check
	poetry run bandit -r ${SRC_DIR} -c pyproject.toml
	poetry run pylint ${SRC_DIR} --rcfile ../.pylintrc
	poetry run isort ${SRC_DIR} --check-only
	poetry run mypy ${SRC_DIR} --show-error-codes
	poetry run radon cc ${SRC_DIR}
	poetry run pytest \
		--disable-warnings \
		--cov-report=html:${TEST_REPORT_DIR} \
		--cov=${SRC_DIR} .
	open ${TEST_REPORT_DIR}/index.html

# Build/Pull infrastructure services
infra-build:
	@echo "Building/Pulling images of infrastructure services..."
	docker-compose -f infrastructure/docker-compose.yml build

# Start infrastructure services
infra-up:
	@echo "Starting infrastructure services..."
	docker-compose -f infrastructure/docker-compose.yml up -d

# Stop infrastructure services
infra-down:
	@echo "Stopping infrastructure services..."
	docker-compose -f infrastructure/docker-compose.yml down
