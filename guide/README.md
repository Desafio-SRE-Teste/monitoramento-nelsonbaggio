# Build Guide

Instruções para gerar o guia com o **claat** (Codelabs command line tool).

## Instalação do Go

[Manual de instalação](https://go.dev/doc/install)

## Instalação do claat

```console
go install github.com/googlecodelabs/tools/claat@latest
```
**Obs.:** Caso o comando acima não funcione, [baixe o binário](https://github.com/googlecodelabs/tools/releases/tag/v2.2.4), renomeie o arquivo baixado para `claat` e mova-o para o diretório `$GOPATH/bin`, que por padrão é `/usr/local/go/bin`.

Por exemplo, se você baixou o arquivo `claat-linux-amd64`, execute os seguintes comandos:
```console
mv claat-linux-amd64 claat
chmod u+x claat
mv claat /usr/local/go/bin/
```

## Build

Na raíz do projeto (diretório sre-academy), execute os seguintes comandos:
```console
claat export -o lab/src guide/*
cp -r lab/src/index/* lab/src/
```
