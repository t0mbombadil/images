.PHONY: tag_check
tag_check:
	@if [ -z "$(IMAGE_TAG)" ]; then\
		echo "IMAGE_TAG variable is undefined!"; \
		exit 1; \
	fi


.PHONY: docker/nginx-build
IMAGE_NAME=base/nginx

IMAGE=$(CONTAINER_REPOSITORY_HOST)/$(IMAGE_NAME):$(IMAGE_TAG)
IMAGE_X86=$(IMAGE)-amd64
IMAGE_ARM=$(IMAGE)-arm64
IMAGE_TARGET=prod # prod or debug

docker/nginx-build: tag_check
  # amd64
	docker buildx build --platform linux/amd64  --no-cache --build-arg ARCH="x86_64" \
		-t $(IMAGE_X86) -f nginx/Dockerfile --target $(IMAGE_TARGET) --load ..
  # arm64
	docker buildx build --platform linux/arm64  --no-cache --build-arg ARCH="aarch64" \
		-t $(IMAGE_ARM) -f nginx/Dockerfile --target $(IMAGE_TARGET) --load ..


.PHONY: docker/nginx-push
docker/nginx-push: tag_check 
	docker push $(IMAGE_X86)
	docker push $(IMAGE_ARM)
  # Currently multi-arch manifest can be created only from pushed images
	docker buildx imagetools create -t $(IMAGE) $(IMAGE_X86) $(IMAGE_ARM)
