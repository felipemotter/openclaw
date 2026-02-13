Fazer commits atomicos das mudancas pendentes.

Regras:

1. Rodar `git status` e `git diff` para ver todas as mudancas.

2. Agrupar mudancas por escopo logico. Cada commit deve ser ATOMICO - uma unica responsabilidade. Exemplos:
   - Mudancas no Dockerfile = um commit
   - Mudancas no docker-compose.yml = outro commit (se nao relacionadas ao Dockerfile)
   - Mudancas em config = outro commit
   - Se Dockerfile e docker-compose.yml mudaram pelo mesmo motivo (ex: adicionar Playwright), podem ir juntos

3. Formato da mensagem de commit (Conventional Commits):
   ```
   tipo(escopo): descricao curta em ingles
   ```
   Tipos: feat, fix, chore, docs, refactor, style, test, ci
   Escopo: area afetada (docker, config, agents, ui, etc)
   Descricao: imperativo, lowercase, sem ponto final, max 72 chars

4. NAO adicionar Co-Authored-By. NAO adicionar nenhum trailer. O unico autor e Felipe Motter <felipe@engenere.one>.

5. NAO usar `git add .` ou `git add -A`. Sempre adicionar arquivos especificos por nome.

6. NAO commitar arquivos com secrets (.env, credentials, tokens, api keys).

7. Para cada grupo de mudancas:
   - Mostrar quais arquivos serao commitados e a mensagem proposta
   - Fazer o commit
   - Seguir para o proximo grupo

8. No final, mostrar `git log --oneline` com os commits criados.
