Atualizar o fork do OpenClaw com o upstream e rebuildar o Docker.

Siga estes passos em ordem:

1. Verificar se tem mudancas pendentes (`git status`). Se tiver, commitar com mensagem descritiva antes de continuar.

2. Buscar atualizacoes do upstream:
   ```
   git fetch upstream
   ```

3. Verificar quantos commits estao atras:
   ```
   git log --oneline HEAD..upstream/main | wc -l
   ```
   Se zero, informar que ja esta atualizado e parar.

4. Rebase no upstream:
   ```
   git rebase upstream/main
   ```
   Se der conflito, informar o usuario e parar.

5. Rebuildar a imagem Docker (o compose nao tem diretiva build, usar docker build direto):
   ```
   docker build --no-cache -t openclaw:local .
   ```

6. Reiniciar os containers:
   ```
   docker compose down && docker compose up -d
   ```

7. Verificar se subiu corretamente:
   ```
   docker compose ps
   ```

8. Tentar push pro fork (pode falhar se auth nao estiver configurada):
   ```
   git push --force-with-lease origin main
   ```

9. Mostrar resumo: quantos commits atualizados, se o build/restart foi ok, e se o push funcionou.
