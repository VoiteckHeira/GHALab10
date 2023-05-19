# Laboratorium 10

GitHub Actions - przeglad podstawowych rozwiazan

Uruchamianym plikiem jest gha_zadanie który odpala sie przy zrobieniu pusha do githuba

w celu recznego uruchomienia nalezy wpisac w konsoli `gh workflow list`, znaleźć interesujący nas workflow

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/ed36034b-fb96-4108-bdc9-bc72cb0a96e4)

wybrać np kod interesujący nas workflow i uruchomić komendą `gh workflow run 5028257381` 

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/0e6a99ec-cd8c-41ad-804f-b85c435ddd03)

podgląd prac mozemy zobaczyć komendą `gh run watch`, gh cli pokaże nam workflowy i możemy wybrać który chcemy podejżeć jak pracuje.

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/efe808c3-a06f-43f4-b411-ffc74a6f4ddd)

lub na stronie

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/02d4eb21-9226-4632-a0ae-d6a81aba6516)


Widok zakończonej pracy

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/9114077d-bdef-43c5-a73a-25198b4460d3)

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/2b9a1ff3-a123-474f-9dc6-39f6d642f021)

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/eb5414f1-4fa9-44c8-ba42-c76b8ea16937)

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/9bc12994-61e0-47c1-b4e6-abae5d091edf)

![image](https://github.com/VoiteckHeira/GHALab10/assets/91530837/31c0888e-8ee1-4049-b251-1236c9095b39)


Plik gha_zadanie.yml
``` yml
name: gha_zad:GitHub Action Zadanie 1

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  build:
    # Definicja systemu do instalacji na weźle roboczym
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3
      # Instalacja środowiska Docker + Buildx
      - name: Buildx set-up
        uses: docker/setup-buildx-action@v2
        with:
          driver-opts: image=moby/buildkit:v0.11.0
          buildkitd-flags: --debug

      # Instalacja QEMU
      - name: Docker Setup QEMU
        uses: docker/setup-qemu-action@v2

      - name: Cache Docker layers
        uses: actions/cache@v3
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      # Logowanie do Dockerhub-a
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{secrets.DOCKERHUB_USERNAME}}
          password: ${{secrets.DOCKERHUB_TOKEN}}

      # Budowa obrazu dla dwóch architektur sprzętowych
      # oraz przesłanie do własnego repo na Dockerhub
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          platforms: linux/arm64/v8,linux/amd64
          context: .
          file: ./Dockerfile_prod
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/gha_lab10:zadv1
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max

        #  tags: |
        #    ${{ secrets.DOCKERHUB_USERNAME }}/gha_lab10:zadv1
        #  cache-from: type=registry,ref=gha_lab10:zadv1
        #  cache-to: type=registry,ref=gha_lab10:zadv1,mode=max
        #  tags: |
        #    ${{ secrets.DOCKERHUB_USERNAME }}/gha_lab10:zadv1
        #  cache-from: type=gha
        #  cache-to: type=gha,mode=max

        # Temp fix
        # https://github.com/docker/build-push-action/issues/252
        # https://github.com/moby/buildkit/issues/1896
      - name: Move cache
        run: |
          rm -rf /tmp/.buildx-cache
          mv /tmp/.buildx-cache-new /tmp/.buildx-cache

```
