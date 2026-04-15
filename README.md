# PAwChO - Laboratorium 6

**Zadanie:** Konfiguracja klienta GitHub CLI, wykorzystanie repozytorium obrazów `ghcr.io` oraz zastosowanie rozszerzonego frontendu (BuildKit) z bezpiecznym wstrzykiwaniem sekretów (SSH).

---

## 1. Treść pliku Dockerfile

Poniżej znajduje się kod pliku `Dockerfile`, który wykorzystuje rozszerzony frontend do bezpiecznego pobrania kodu z prywatnego repozytorium za pomocą kluczy SSH podłączonych w trakcie budowy.

```dockerfile
# syntax=docker/dockerfile:1.2-labs

# ==========================================
# STAGE 1: Budowanie
# ==========================================
FROM alpine AS builder

RUN apk add --no-cache openssh-client git
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

# Klonowanie repozytorium z wykorzystaniem udostępnionego agenta SSH
RUN --mount=type=ssh \
    git clone git@github.com:k4p1-PL/pawcho6.git /app

# ==========================================
# STAGE 2: Serwer Nginx
# ==========================================
FROM nginx:alpine

COPY --from=builder /app /usr/share/nginx/html

HEALTHCHECK --interval=10s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

EXPOSE 80
```


## 2. Budowa obrazu i wynik działania

### docker build --ssh default -t lab6

PS C:\Users\kacpe\lab5> docker build --ssh default -t lab6 .
[+] Building 18.9s (13/13) FINISHED                                                                docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 1.22kB                                                                             0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                   2.1s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                    3.1s
 => [auth] library/nginx:pull token for registry-1.docker.io                                                       0.0s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                      0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [builder 1/4] FROM docker.io/library/alpine:latest@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20ac  0.1s
 => => resolve docker.io/library/alpine:latest@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c  0.0s
 => [stage-1 1/2] FROM docker.io/library/nginx:alpine@sha256:582c496ccf79d8aa6f8203a79d32aaf7ffd8b13362c60a701a2f  6.0s
 => => resolve docker.io/library/nginx:alpine@sha256:582c496ccf79d8aa6f8203a79d32aaf7ffd8b13362c60a701a2f9ac64886  0.0s
 => => sha256:1165b869c51a1a0747d78cec8fab96c30156a979e51ecf2f91aa792e557d94a4 20.25MB / 20.25MB                   3.1s
 => => sha256:34dfdd2ef1f920d0054dde2fc09ddc83ff8e71d05fadb79e2cab6e6234596f0a 1.21kB / 1.21kB                     0.5s
 => => sha256:c8a2fa3a88d244a3f32dcbc9c1f7649c662661a28c624198ada43aa0b7598e7f 1.40kB / 1.40kB                     0.6s
 => => sha256:a71873b303e8d75170b7ced2725b01b3ae15ad76f0d4eef16a49335821b6a0ef 404B / 404B                         0.7s
 => => sha256:ff9f59a6a62e9e9f29d7a84fb18865b45664d3f0d061eff7548bd61746dd101c 957B / 957B                         0.7s
 => => sha256:15e759724ff67f262e38bb7c070af9d0b84f959f9b37fa966f68bf2f881a4b62 627B / 627B                         0.7s
 => => sha256:f03becc8ac15611cfcc421c977a5ba4d65456093570788523a4ba557689aa7f7 1.87MB / 1.87MB                     3.9s
 => => extracting sha256:f03becc8ac15611cfcc421c977a5ba4d65456093570788523a4ba557689aa7f7                          0.3s
 => => extracting sha256:15e759724ff67f262e38bb7c070af9d0b84f959f9b37fa966f68bf2f881a4b62                          0.0s
 => => extracting sha256:ff9f59a6a62e9e9f29d7a84fb18865b45664d3f0d061eff7548bd61746dd101c                          0.0s
 => => extracting sha256:a71873b303e8d75170b7ced2725b01b3ae15ad76f0d4eef16a49335821b6a0ef                          0.0s
 => => extracting sha256:34dfdd2ef1f920d0054dde2fc09ddc83ff8e71d05fadb79e2cab6e6234596f0a                          0.0s
 => => extracting sha256:c8a2fa3a88d244a3f32dcbc9c1f7649c662661a28c624198ada43aa0b7598e7f                          0.0s
 => => extracting sha256:1165b869c51a1a0747d78cec8fab96c30156a979e51ecf2f91aa792e557d94a4                          0.7s
 => [builder 2/4] RUN apk add --no-cache openssh-client git                                                        8.4s
 => [builder 3/4] RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts                      2.2s
 => [builder 4/4] RUN --mount=type=ssh     git clone git@github.com:k4p1-PL/pawcho6.git /app                       3.9s
 => [stage-1 2/2] COPY --from=builder /app /usr/share/nginx/html                                                   0.2s
 => exporting to image                                                                                             0.7s
 => => exporting layers                                                                                            0.4s
 => => exporting manifest sha256:157e9c6035e5dbf17068c06bc1bf10ee7e520ab1f972b7e9f75b1fe6b6218630                  0.0s
 => => exporting config sha256:a70dc7df04b4c894bb73959d6bf909b92f3ff79ed0d97c5f1944974deefbb524                    0.0s
 => => exporting attestation manifest sha256:97d8b91a6f6f0a6419923bc0a5fddbc4a0090d779aae82e4f5f125fefa7c1933      0.0s
 => => exporting manifest list sha256:42784f12a4bcf8d8a914f7a7ff4a2b6a9f35ecfc31b06a598b2adf74242f1253             0.0s
 => => naming to docker.io/library/lab6:latest                                                                     0.0s
 => => unpacking to docker.io/library/lab6:latest                                                                  0.1s

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/x93ljkfa2bcm932y6ver0put1


### echo "token" | docker login ghcr.io -u k4p1-pl --password-stdin

PS C:\Users\kacpe\lab5> echo "token" | docker login ghcr.io -u k4p1-pl --password-stdin

Login Succeeded

### docker tag lab6 ghcr.io/k4p1-pl/pawcho6:lab6


### docker push ghcr.io/k4p1-pl/pawcho6:lab6

### docker push ghcr.io/k4p1-pl/pawcho6:lab6

The push refers to repository [ghcr.io/k4p1-pl/pawcho6]