RUNTIME = python3.11
REGION = us-east-1

LAYER_NAME = pdf-libs-layer
DESCRIPTION = "Python3.11: aws-lambda-powertools (2.14.1), aws-xray-sdk (2.13.0), requests, xsdata (24.11), PyJWT (2.9.0), cffi (1.17.1), regex, certifi, charset-normalizer, wrapt, gspread, oauth2client, fuzzywuzzy, PyMuPDF (1.23.26)"

.PHONY: all build run clean deploy


all: build clean

build:
	@echo "Building Docker image..."
	docker build --platform linux/amd64 -t $(LAYER_NAME) .
	docker run --rm --platform linux/amd64 \
		-v "$(PWD)":/var/task \
		--entrypoint /bin/bash \
		$(LAYER_NAME) \
		-c "cp /opt/$(LAYER_NAME).zip /var/task/ && chown $(shell id -u):$(shell id -g) /var/task/$(LAYER_NAME).zip"
	@echo "Layer created: $(LAYER_NAME).zip"

test:
	docker compose up --build

run:
	docker run -it --rm --platform linux/amd64 \
		-v "$(PWD)":/var/task \
		--entrypoint /bin/bash \
		$(LAYER_NAME)

clean:
	@echo "Cleaning up..."
	-docker rmi $(LAYER_NAME) 2>/dev/null || true
	@echo "Cleanup complete!"

publish:
	@echo "Deploying $(LAYER_NAME).zip to AWS Lambda..."
	@VERSION=$$(aws lambda publish-layer-version \
		--layer-name $(LAYER_NAME) \
		--zip-file fileb://$(LAYER_NAME).zip \
		--compatible-runtimes $(RUNTIME) \
		--region $(REGION) \
		--description "$$(echo $(DESCRIPTION) | sed 's/"/\\"/g')" \
		--output json | jq -r '.Version'); \
	aws lambda add-layer-version-permission \
#		--layer-name $(LAYER_NAME) \
#		--statement-id grant-access-to-dev-$$VERSION \
#		--principal 000000000000 \
#		--action lambda:GetLayerVersion \
#		--version-number $$VERSION;  \
	echo "Published version: $$VERSION"

