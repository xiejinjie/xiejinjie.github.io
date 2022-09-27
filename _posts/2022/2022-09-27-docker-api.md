---
layout: post
title: "docker API使用"
date: 2022-09-27 09:22:00 +0800
categories: tech
---
## Docker API简介

Docker提供了Docker Engine API与Docker daemon交互，Docker提供了Go和Python版本的SDK。本质上Docker Engine API是一个RESTful的接口，可以通过HTTP请求操作。

![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/Docker-build-900x551.png.webp)

这里以GO版本的docker client演示拉取镜像、创建容器、启动容器、停止容器、删除容器、容器状态监听 一系列操作API的示例。全部的api文档：[点击这里](https://pkg.go.dev/github.com/docker/docker/client)

## 获取client

通过client提供的方法来实现对Docker的操作

```
cli, err := client.NewClientWithOpts(client.FromEnv)
```

## 镜像操作

- 查看本地镜像列表:

```
func listImages(cli *client.Client) {
	images, err := cli.ImageList(context.Background(), types.ImageListOptions{})
	if err != nil {
		panic(err)
	}
	for _, image := range images {
		fmt.Printf("%s %s \n\n", image.ID[:19], image.RepoTags)
	}
}
```

- 拉取新的镜像：

```
func ImagePull(cli *client.Client) {
	dockerImage := "docker/getting-started"
	reader, err := cli.ImagePull(context.Background(), dockerImage, types.ImagePullOptions{})
	if err != nil {
		panic(err)
	}
	io.Copy(os.Stdout, reader)
}
```

- 删除镜像：
  func ImageRemove(cli *client.Client) {
  imageID := "cb90f98fd791"
  items, err := cli.ImageRemove(context.Background(), imageID, types.ImageRemoveOptions{})
  if err != nil {
  panic(err)
  }
  for _, item := range items {
  fmt.Printf("%s %s\n\n", item.Deleted, item.Untagged)
  }
  }

## 容器操作

- 查看当前运行的容器列表：

```
func listContainers(cli *client.Client) {
	containers, err := cli.ContainerList(context.Background(), types.ContainerListOptions{})
	if err != nil {
		panic(err)
	}

	for _, container := range containers {
		fmt.Printf("%s %s\n\n", container.ID[:10], container.Image)
	}
}
```

- 根据镜像创建容器：

```
func ContainerCreate(cli *client.Client) string {
	containerName := "docker-demo"
	containerPort, _ := nat.NewPort("tcp", "80")
	hostBinding := nat.PortBinding{
		HostIP:   "0.0.0.0",
		HostPort: "80",
	}
	cont, err := cli.ContainerCreate(context.Background(),

		&container.Config{
			Image: "docker/getting-started",
		},
		&container.HostConfig{
			PortBindings: nat.PortMap{
				containerPort: []nat.PortBinding{
					hostBinding,
				},
			},
		},
		nil, nil, containerName)
	if err != nil {
		panic(err)
	}
	fmt.Printf("containerId = %s \n\n", cont.ID)
	return cont.ID
}
```

- 启动容器：

```
func ContainerStart(cli *client.Client, cId string) {
	err := cli.ContainerStart(context.Background(), cId, types.ContainerStartOptions{})
	if err != nil {
		panic(err)
	}
}
```

- 删除容器：

```
func ContainerRemove(cli *client.Client) {
	containerName := "docker-demo"
	err := cli.ContainerRemove(context.Background(), containerName, types.ContainerRemoveOptions{})
	if err != nil {
		panic(err)
	}
}
```

- 容器状态监听：

```
func MonitorContainers(cli *client.Client, containerID string) {
	statusCh, errCh := cli.ContainerWait(
		context.Background(),
		containerID,
		container.WaitConditionNotRunning,
	)

	select {
	case err := <-errCh:
		if err != nil {
			log.Fatal(err)
		}
	case status := <-statusCh:
		fmt.Printf("status code = %d\n", status.StatusCode)
	}
}
```

[代码在这里可以查看](https://github.com/xiejinjie/demo/blob/main/demo-docker-api/main.go)
