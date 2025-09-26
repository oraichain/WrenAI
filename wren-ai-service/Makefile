# Color definitions
GREEN := \033[32m
YELLOW := \033[33m
RESET := \033[0m

# Default values
dev ?= --dev
dataset ?= spider1.0
pipeline ?= ask
semantics ?= --no-semantics
usecases ?= all
lang ?= en
test_args ?=
mdl_path ?=
data_source ?=
sample_dataset ?=

.PHONY: init up down start curate_eval_data prep predict eval test test-usecases load-test prepare-files force_update_config run-sql mdl-to-str

## todo: consider to support --override flag to override existing files
init:
	@if [ ! -f config.yaml ]; then \
		echo "$(GREEN)config.yaml does not exist. Creating from example...$(RESET)"; \
		cp tools/config/config.example.yaml config.yaml; \
	else \
		echo "$(YELLOW)config.yaml already exists. Skipping creation.$(RESET)"; \
	fi 

	@if [ $(dev) = "--dev" ] || [ $(dev) != "--non-dev" ]; then \
		if [ ! -f .env.dev ]; then \
			echo "$(GREEN).env.dev does not exist. Creating from example...$(RESET)"; \
			cp tools/config/.env.dev.example .env.dev; \
		else \
			echo "$(YELLOW).env.dev already exists. Skipping creation.$(RESET)"; \
		fi \
	fi

up: prepare-files
	docker compose -f ./tools/dev/docker-compose-dev.yaml --env-file ./tools/dev/.env up -d

down:
	docker compose -f ./tools/dev/docker-compose-dev.yaml --env-file ./tools/dev/.env down

start: force_update_config
	poetry run python -m src.__main__

curate_eval_data:
	poetry run streamlit run eval/data_curation/app.py

prep:
	poetry run python -m eval.preparation --dataset $(dataset)

predict:
	poetry run python -u eval/prediction.py --file $(dataset) --pipeline $(pipeline)

eval:
	poetry run python -u eval/evaluation.py --file $(prediction_result) $(semantics)

test: up down
	poetry run pytest -s $(test_args) --ignore tests/pytest/test_usecases.py

test-usecases:
	poetry run python -m tests.pytest.test_usecases --usecases $(usecases) --lang $(lang)

load-test:
	poetry run python -m tests.locust.locust_script

prepare-files:
	# only remove files related to engine and ui
	rm -rf tools/dev/etc/duckdb tools/dev/etc/mdl tools/dev/etc/config.properties tools/dev/etc/db.sqlite3 tools/dev/etc/archived
	mkdir -p tools/dev/etc
	cp tools/dev/config.properties.example tools/dev/etc/config.properties
	mkdir -p tools/dev/etc/mdl
	echo '{"catalog": "test_catalog", "schema": "test_schema", "models": []}' \
		> tools/dev/etc/mdl/sample.json

force_update_config:
	poetry run python -m src.force_update_config

run-sql:
	poetry run python tools/run_sql.py --mdl-path "$(mdl_path)" --data-source "$(data_source)" --sample-dataset "$(sample_dataset)"

mdl-to-str:
	poetry run python tools/mdl_to_str.py -p $(mdl_path)