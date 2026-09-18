---
name: generate-pull-request
description: Gera e cria um Pull Request em draft para a branch atual, comparando com uma branch alvo (TARGET), preenchendo o template do repositório com base nos commits e no diff.
---

# Generate Pull Request

Siga os passos abaixo **na ordem**, sem pular validações. Cada passo tem uma condição de parada explícita — se ela ocorrer, pare e reporte ao usuário em vez de tentar contornar.

## 1. Resolver `TARGET`

`TARGET` é o nome da branch de destino do PR (ex.: `main`).

- Se o usuário já informou a branch alvo nesta conversa ou como argumento da skill, use-a como `TARGET`.
- Caso contrário, **pare** e peça explicitamente: "Para qual branch devo abrir o PR? (TARGET)". Nunca assuma `main`/`master` por padrão.

Todas as referências a `TARGET` abaixo usam exatamente o valor resolvido aqui (substitua o literal, não passe a string `TARGET` para os comandos).

## 2. Validar `TARGET`

```
git rev-parse --verify origin/TARGET
```

- Se o `rev-parse` falhar, **pare**: a branch não existe. Reporte o erro ao usuário.
- Se `TARGET` for igual à branch atual, **pare**: não há o que comparar.

## 3. Levantar commits da branch

```
git log origin/TARGET..HEAD --oneline
```

- Se a saída for vazia, **pare**: não há commits novos em relação a `TARGET`, não faz sentido abrir PR.
- Guarde a lista completa de commits (com corpo, se necessário) para extrair contexto: `git log origin/TARGET..HEAD`.

## 4. Levantar o diff

Use três pontos (diff a partir do merge-base, o que reflete o que o PR realmente vai propor), não dois:

```
git diff origin/TARGET...HEAD --stat
git diff origin/TARGET...HEAD
```

Use o diff junto com as mensagens de commit para entender o contexto real das mudanças (não invente contexto que não esteja em um ou outro).

## 5. Ler o template de PR

O template fica em `.github/pull_request_template.md`. Leia o conteúdo real do arquivo — não presuma a estrutura.

## 6. Gerar o título (Conventional Commits, determinístico)

Regras para escolher o `type`:

1. Se todos os commits da branch usam o mesmo tipo (prefixo antes de `:` ou `(`), use esse tipo.
2. Se houver mistura, use esta prioridade (do mais para o menos significativo): `feat` > `fix` > `refactor` > `chore` > `tests` > `docs`.
3. `scope`: use o escopo mais frequente entre os commits (ex.: `nfse`, `regra-tributacao`). Se não houver escopo consistente, omita o `()`.
4. Descrição: resumo objetivo do que o diff faz, no imperativo, minúsculo, sem ponto final, máximo ~72 caracteres no total do título.

Formato final: `tipo(escopo): descrição objetiva`

## 7. Preencher o template

Gere um arquivo markdown temporário em um diretório de scratch (não no repositório) preenchendo **cada seção do template lido no passo 8** com base apenas em fatos extraídos do log de commits e do diff:

> Nunca deixe placeholders/exemplos do template (como `GET /wisedoc/novo-endpoint`) no arquivo final caso não haja endpoint correspondente real.

## 8. Criar o PR

Com `TARGET` e `titulo` já resolvidos, e `arquivo.md` sendo o caminho gerado no passo 10:

```
gh pr create --draft --title "<titulo>" --body-file "<arquivo>.md" --base "<TARGET>"
```

- Interpole os valores reais — nunca envie a string literal `TARGET` como argumento de `--base`.
- Após a criação, capture a URL retornada por `gh pr create` e reporte ao usuário. Não rode `gh pr view --web` nem abra nada automaticamente sem pedir.

## Condições de parada (resumo)

Pare e reporte ao usuário, sem tentar contornar, se:
- `TARGET` não foi informado nem pode ser inferido do contexto já dado pelo usuário.
- `TARGET` não existe ou é igual à branch atual.
- Não há commits novos em relação a `TARGET`.
- O `gh` CLI não está autenticado (`gh auth status` falha).
