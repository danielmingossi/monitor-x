# monitor_x

Skill do [Claude Code](https://claude.com/claude-code) que procura no X (Twitter), pelo seu próprio navegador, postagens da última hora com **ameaça, calúnia, difamação ou desinformação dirigidas contra** pessoas que você quer proteger. Devolve um relatório só com os casos que se enquadram nos critérios do canal [vigiarepunir.com](https://vigiarepunir.com/), com link, trecho, enquadramento, grau de confiança e o texto pronto para o campo "Contexto" do formulário.

A skill não envia nada. Quem revisa e decide denunciar é você.

## Princípio

A busca parte da **vítima**, nunca do autor. Não serve para vigiar contas nem listas de pessoas por ideologia. Crítica, opinião, sátira e notícia sobre investigação real não são denunciáveis. Os critérios estão em [`referencia/criterios.md`](referencia/criterios.md).

## Requisitos

- Claude Code com a extensão **Claude in Chrome** conectada.
- Chrome logado no X.
- Sem API do X e sem custo extra além do seu plano Claude.

## Instalação

```bash
git clone https://github.com/danielmingossi/monitor-x.git ~/.claude/skills/monitor_x
```

Crie a pasta de trabalho (`D:\Claude\Monitor X` no Windows com unidade D:, ou `~/Monitor X`) com um arquivo `alvos.txt`, um perfil por linha:

```
@FlavioBolsonaro | Flávio Bolsonaro
```

## Uso

- Uma rodada: `/monitor_x`
- Repetir a cada hora com o computador ligado: `/loop 1h /monitor_x`

Os relatórios ficam em `relatorios/` na pasta de trabalho.

## Limitações

- A busca do X não mostra tudo. Em perfis com muito volume, a rodada cobre só parte da hora.
- Postagens só com imagem ou vídeo não são avaliadas.
- O print para a denúncia é manual.
- Automação no X pode contrariar os termos da plataforma. Mantenha o intervalo mínimo de 1 hora.
