id:     load-test
Title:  SRE Academy - Teste de Carga

# SRE Academy - Teste de Carga

## Teste de Carga

- Acesse pelo seu terminal o diretório do projeto e acesse a pasta "load-gen"
- Execute o arquivo load-gen.sh

``` console
./load-gen.sh
```

**Obs.:** É possível alterar os parâmetros do teste. Segue abaixo os parâmetros existentes:<br>
`e` - error flag<br>
`d` - run in background<br>
`n` - number of clients<br>
`t` - time to run n clients<br>
`h` - target host

**Ex.:**
``` console
./load-gen.sh -n 50 -t 5m
```
O comando acima executa o teste de carga simulando 50 usuários pelo período de 5 minutos.
