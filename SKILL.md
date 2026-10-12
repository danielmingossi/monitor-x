---
name: monitor_x
description: Use when the user wants to monitor X (Twitter) for threats, calúnia, difamação or fake news directed AT a protected person listed in alvos.txt, in the last hour, and get a report of only the posts that qualify for a denúncia at vigiarepunir.com. Keywords monitor X, monitorar ataques, ameaças contra, denúncia vigiar e punir, varredura do X, rodar monitor. Uses the user's own Chrome (Claude in Chrome), no X API.
---

# Monitor X (ataques contra perfis protegidos)

Varre o X pelo navegador do próprio usuário (já logado no X), procura postagens da última hora **dirigidas contra** os perfis da lista de alvos e devolve só os casos que se enquadram nos critérios de denúncia do site vigiarepunir.com. A decisão de enviar é sempre do usuário; a skill nunca preenche nem envia o formulário.

## Princípio que não muda

O ponto de partida é a **vítima**, nunca o autor. Não monte buscas a partir de listas de autores, e não analise contas inteiras a pedido; a ideologia de quem posta não pesa na triagem. Crítica, opinião, sátira evidente e notícia sobre investigação ou processo público **não** são denunciáveis, por mais duras que sejam. Na dúvida, o caso vai para "Descartados" ou entra com confiança baixa.

## Arquivos

Pasta de trabalho: `D:\Claude\Monitor X`; se a unidade D: não existir, `~/Monitor X`. Criar se não existir.

- `alvos.txt`: um perfil por linha no formato `@usuario | Nome por extenso`. Se não existir, perguntar ao usuário quem proteger.
- `vistos.txt`: URLs de postagens já avaliadas (uma por linha), para não reavaliar entre rodadas.
- `relatorios/AAAA-MM-DD_HHMM.md`: um relatório por rodada.

Critérios de enquadramento: `referencia/criterios.md` desta skill. Ler antes da triagem.

## Rodada

1. **Preparar.** Ler `alvos.txt` e `vistos.txt`. Janela fixa de 1 hora.
2. **Abrir o navegador.** Carregar a skill `claude-in-chrome` e as ferramentas numa única ToolSearch (`tabs_context_mcp`, `navigate`, `javascript_tool`, `tabs_close_mcp`). Chamar `tabs_context_mcp` com `createIfEmpty` e usar a aba do grupo. Se o X pedir login, parar e avisar.
3. **Buscar.** Para cada alvo, só estas 2 buscas, na aba "Mais recentes" (`&f=live`):
   - `"Nome por extenso" (morrer OR matar OR tiro OR bala OR enforcar OR forca OR cadeia OR ladrão OR bandido OR miliciano) within_time:1h`
   - `"Nome por extenso" -from:usuario within_time:1h`

   URL: `https://x.com/search?q=<consulta codificada>&src=typed_query&f=live`. O operador `within_time` funcionou no teste de 12/10/2026; se falhar, usar `since:AAAA-MM-DD_HH:MM:SS_UTC`.
4. **Ler.** `get_page_text` não serve (devolve só uma postagem). Usar `javascript_tool` com o coletor abaixo, que rola e acumula num `Map` porque o X descarta da página as postagens já roladas.
   - Uma chamada não pode passar de 45 s: no máximo 10 rolagens de 1.500 px com espera de 2,5 s.
   - Não devolver `location.href` nem URL com query string (a extensão bloqueia o retorno).
   - O retorno é cortado perto de 1.000 caracteres: as linhas ficam em `window.__lines`; ler em fatias (`window.__lines.slice(n).join('\n').slice(0,950)`).
   - Informar no relatório a cobertura real (horário da postagem mais antiga lida em cada busca).
   - Postagens só com imagem ou vídeo (marcadas `[m]` sem texto) não são avaliadas; contar no relatório.

   ```js
   await new Promise(r=>setTimeout(r,3000)); window.__m=new Map();
   const grab=()=>document.querySelectorAll('article[data-testid="tweet"]').forEach(a=>{const t=a.querySelector('time');const l=t?.closest('a')?.getAttribute('href');if(!l||window.__m.has(l))return;
     const txt=a.querySelector('[data-testid="tweetText"]')?.innerText||'';const media=a.querySelector('[data-testid="tweetPhoto"],video')?' [m]':'';
     window.__m.set(l,t.getAttribute('datetime').slice(11,16)+' | '+txt.replace(/\n/g,' ').slice(0,160)+media)});
   for(let i=0;i<10;i++){grab();window.scrollBy(0,1500);await new Promise(r=>setTimeout(r,2500));} grab();
   window.__lines=[...window.__m.entries()].map(([k,v])=>k.slice(1).replace('/status/',' ')+' | '+v);
   window.__lines.length+'\n'+window.__lines.join('\n').slice(0,950)
   ```
   Horários vêm em UTC; converter para o horário de Brasília (UTC−3). Ignorar URLs presentes em `vistos.txt`.
5. **Triar.** Aplicar `referencia/criterios.md` a cada postagem. Abrir a postagem individual só quando o contexto (thread, imagem, vídeo) for indispensável para decidir. Para candidato a calúnia ou desinformação, fazer uma WebSearch para checar se o fato ou a frase existe; no máximo uma busca por caso.
6. **Gravar.** Escrever o relatório (modelo abaixo) e acrescentar ao `vistos.txt` todas as URLs avaliadas, positivas ou não. Fechar a aba usada.
7. **Responder** em uma linha com o número de positivos por categoria e o caminho do relatório.

Ritmo: 2 buscas por alvo por rodada. Em repetição, usar `/loop 1h /monitor_x` (intervalo mínimo de 1 hora) para não expor a conta do usuário a bloqueio por automação.

## Modelo do relatório

```
# Monitor X, rodada de DD/MM/AAAA HH:MM (janela de 1 h)

Alvos: @... | Postagens avaliadas: N | Positivas: N | Descartadas: N
Cobertura: busca 1 até HH:MM, busca 2 até HH:MM | Só mídia, não avaliadas: N

## Positivos

### 1. [Categoria] confiança alta|média|baixa
- Link: https://x.com/usuario/status/...
- Autor: Nome (@usuario), publicado em DD/MM/AAAA HH:MM
- Trecho: "texto literal"
- Enquadramento: por que se encaixa no critério, em 1 a 3 frases
- Contexto para o formulário: texto pronto para colar no campo 03 (até 2.000 caracteres)
- Print: tirar manualmente mostrando perfil, data e texto completos, sem recorte

## Descartados
Uma linha por postagem: link + motivo curto (crítica, opinião, notícia, sátira etc.).
```

O texto "Contexto para o formulário" é sóbrio: sem travessão, sem dois pontos no meio de frase, sem clichês. Informar onde a postagem apareceu, quando, contra quem e por que é ameaça, calúnia, difamação ou desinformação.
