# Docker picgo typora图床

[官网](https://docs.min.io/docs/minio-docker-quickstart-guide.html)

```sh
docker run \
  -p 9000:9000 \
  -p 9001:9001 \
  --name minio \
  -v /mydata/minio/data:/data \
  -v /mydata/minio/config:/root/.minio \
  -e "MINIO_ROOT_USER=zjm" \
  -e "MINIO_ROOT_PASSWORD=zjm17734808250" \
  quay.io/minio/minio server /data --console-address ":9001"
```

