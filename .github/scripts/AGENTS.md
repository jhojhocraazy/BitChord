# AGENTS.md — Diretriz local de localização: BitChord

## 1. Domínio

Player de música nativo Android em Kotlin Multiplatform + Compose, com Material 3, alvos phone, TV e
Wear. O diferencial do produto é a cadeia de áudio de alta fidelidade: reprodução sem perdas, saída
exclusiva por USB para DAC externo, transições entre faixas sem silêncio e processamento de sinal
opcional.

O usuário típico investiu em fone e DAC e reconhece jargão de áudio. Ele estranha tanto a paráfrase
que desmancha um termo consagrado quanto o inglês desnecessário. O ponto de equilíbrio não se adivinha:
se descobre pela evidência da comunidade.

## 2. Onde o texto vive (verificado no clone, revisado)

Existem **três** catálogos de string no repositório, não um. O `sharedUi` não é o único dono do texto:
o `app` é o maior catálogo do projeto e o `wearos` tem o seu próprio.

| Caminho | Papel | Inventário verificado |
| --- | --- | --- |
| `sharedUi/src/commonMain/composeResources/values/strings.xml` | fonte EN do compose (UI compartilhada, player, letras, mix) | 250 `<string>`, 2 `<string-array>` (24 `<item>`), 0 `<plurals>` |
| `sharedUi/src/commonMain/composeResources/values-pt/strings.xml` | **alvo** — pt-BR, espelho integral | 250 `<string>`, 2 `<string-array>` (24 itens) |
| `app/src/main/res/values/strings.xml` | fonte EN do app Android (ajustes, fontes, WebDAV/SMB, Discord, bit-perfect, Replay, Listen Together, Wear OS) | 1.169 `<string>` = **1.137 traduzíveis + 32 com `translatable="false"`**; 23 `<plurals>` (46 `<item>`) |
| `app/src/main/res/values-pt/strings.xml` | **alvo** — pt-BR, espelho integral | 1.169 `<string>` (as 32 não traduzíveis copiadas verbatim), 23 `<plurals>` |
| `wearos/src/main/res/values/strings.xml` | fonte EN do app de relógio | 40 `<string>`, 0 `<plurals>`, 0 `<string-array>` |
| `wearos/src/main/res/values-pt/strings.xml` | **alvo** — pt-BR | 40 `<string>` |

Totais: **1.459 chaves `<string>`** (1.427 traduzíveis + 32 `translatable="false"`), **23 `<plurals>`**
com 46 itens e **24 itens de `<string-array>`**. Desses, **90 nomes de chave existem ao mesmo tempo em
`app` e `sharedUi`** (catálogo de shelf/mood/biblioteca/player): o texto traduzido precisa ser
**idêntico nos dois arquivos**, senão a mesma tela mostra rótulos diferentes conforme o módulo que a
renderiza.

### Diretório alvo: `values-pt`, não `values-pt-rBR`

O seletor nativo de idioma do app resolve a variante brasileira pelo diretório `values-pt` já
existente. Criar `values-pt-rBR` exigiria mudar código Kotlin, e uma variante `pt-rBR` parcial
sombrearia parte de `pt` e herdaria o resto — mistura de duas traduções na mesma tela. Portanto:

- o alvo é **`values-pt`** nos três módulos;
- **`values-pt-rBR` não é criado** e não deve ser criado enquanto o seletor não for alterado;
- `sharedUi` tem 18 diretórios de locale (incluindo `pt`); `app` tem 19 (incluindo `pt`, mais os
  qualificadores não-idioma `values-night` e `values-v31`); `wearos` tem só `values` e `values-pt`;
- `tv/`, `shared/` e `web/` não possuem catálogo de string — não recebem tradução.

## 3. Protocolo de inferência semântica

1. **Rastrear até o catálogo dono.** `sharedUi` (Compose) ou `app`/`wearos` (Android res). Se a string
   não existe em nenhum catálogo EN, o problema é arquitetural: reporte, não invente chave.
2. **Ler o contexto de renderização.** Qual componente consome? É `Text` de botão, item de lista,
   `supportingText`? O limite de caracteres vem do slot, não do bom senso.
3. **Classificar o vocabulário em três faixas:**
   - *jargão de áudio* — o público já usa em inglês ou em empréstimo consagrado; verifique como fóruns
     brasileiros de áudio e apps concorrentes escrevem antes de decidir;
   - *interface comum* — deve seguir o vocabulário do Android em pt-BR, não invenção própria;
   - *nome de recurso do produto* — trata-se como nome próprio.
4. **Checar simetria de gênero e número.** Configurações de áudio erram feio em "o taxa" / "a ganho".
   Confirme o gênero pelo uso real, não pela terminação inglesa.
5. **Registrar a evidência.** Cada termo de jargão entra no handoff (§8) com a fonte consultada.

## 4. Invariantes técnicas

- `%1$s`, `%2$d`, `%%`, `%d` e ordem de argumentos preservados — inclusive dentro de `<plurals>`.
- Apóstrofo em XML Android escapado (`\'`); `&` como `&amp;`; `<` como `&lt;`.
- `name` idêntico ao `values/strings.xml`; nenhuma chave nova sem par em EN; ordem dos elementos igual
  à do EN.
- `<plurals>`: manter exatamente as mesmas `quantity` do EN (`one`/`other`) e o mesmo número de itens;
  `<string-array>`: mesmo número de `<item>`, na mesma ordem.
- As 32 chaves com `translatable="false"` do `app` (código de testador, nomes de ícone, `PCM Data Rate`,
  preferência de idioma do import do Spotify) são copiadas **verbatim**, com o atributo preservado.
- As 90 chaves duplicadas `app`↔`sharedUi` têm texto byte a byte idêntico.
- Arquivos em **UTF-8 sem BOM, CRLF**, com comentários `<!-- -->` do EN preservados.
- Nomes de formato (`FLAC`, `ALAC`, `Opus`, `LDAC`, `aptX`), marcas (`Android`, `Wear OS`,
  `Chromecast`, `YouTube Music`, `Spotify`, `Discord`) e nomes de campo de API não se traduzem.

## 5. Critérios de aceitação

- Botão ≤ 24 caracteres; item de lista ≤ 40; texto de apoio ≤ 100 (títulos de configuração e avisos
  longos podem passar — o slot é a régua).
- Caixa de frase (Material 3 não usa Title Case); ação no infinitivo.
- Sem ponto final em rótulo de botão, item de menu ou título de seção.
- Unidades no padrão brasileiro: "24 bits" (sem hífen), "%1$d canais" (não "ch"), "%1$d compassos"
  (não "bars"), "%1$d s", "%1$d BPM".
- O termo de áudio escolhido precisa ser reconhecível por quem lê review de DAC — teste de
  reconhecimento, não de dicionário.
- Nenhuma chave criada fora dos três catálogos; nenhuma string de UI em Kotlin.

## 6. Proibições

- Traduzir jargão de áudio por paráfrase explicativa quando a comunidade usa a forma curta.
- Reaproveitar o `values-pt` anterior como referência: ele era incompleto (170 de 250 chaves no
  `sharedUi`) e de variante não-brasileira. A tradução é refeita do EN.
- Criar `values-pt-rBR`, ou qualquer locale novo, sem alterar o seletor em Kotlin.
- Traduzir nome de formato, marca ou identificador.
- Usar gerúndio em ação, Title Case, ponto final em botão ou segunda pessoa ("teu", "tua", "contigo").
- Divergir o texto das 90 chaves duplicadas entre `app` e `sharedUi`.

## 7. Verificação

```powershell
# integridade XML dos três catálogos (sem SDK Android instalado, esta é a régua disponível)
[xml](Get-Content -Raw 'sharedUi/src/commonMain/composeResources/values-pt/strings.xml') | Out-Null
[xml](Get-Content -Raw 'app/src/main/res/values-pt/strings.xml') | Out-Null
[xml](Get-Content -Raw 'wearos/src/main/res/values-pt/strings.xml') | Out-Null
```

```bash
# com SDK disponível
./gradlew :sharedUi:compileDebugKotlinAndroid :app:processDebugResources :wearos:assembleDebug
```

Checklist: paridade de `name` e de ordem, placeholders, `\n`, `%%`, escape, gênero, limite de
caracteres, uniformidade das 90 duplicadas, `translatable="false"` intactas, CRLF sem BOM.

## 8. Handoff

| chave | EN | pt-BR escolhido | faixa | evidência | alternativa rejeitada | motivo |
| --- | --- | --- | --- | --- | --- | --- |
| `lossless`, `hi_res_lossless` | Lossless / Hi-Res Lossless | Lossless / Hi-Res Lossless | jargão (selo) | [ABRAMUS — Diferenças do HiFi](https://www.abramus.org.br/noticias/19507/diferencas-do-hifi/); [AAT Audio — fontes de sinal](https://aataudio.com.br/blog/wp-content/uploads/fontes-de-sinal-de-audio-5.pdf) | "Sem perdas" / "Alta resolução" | é selo de qualidade exibido como badge; o texto corrido brasileiro usa "sem perdas", mas o selo aparece em comparação de formatos e o usuário o procura pelo nome |
| `bit_perfect_output`, `pipeline_bit_exact`, `bit_perfect_status_*` | Bit-perfect / Bit-exact | bit-perfect / Bit-exact | jargão | [Neutron — Bit-Perfect](https://neutroncode.com/feature_bit-perfect) | "Bits inalterados" | descreve exatamente o bits inalterados até o DAC; é o termo que o público usa ao avaliar player |
| `prefer_usb_dac`, `dac_volume`, `bit_perfect_mode_*` | DAC | DAC | jargão | [Neutron — Bit-Perfect](https://neutroncode.com/feature_bit-perfect); [iFi — Bit-Perfect DSD](https://ifiaudio.jp/%e8%a6%81%e7%b4%a0%e6%8a%80%e8%a1%93-%e3%82%af%e3%82%a2%e3%83%89%e3%83%bb%e3%82%b9%e3%82%bf%e3%83%83%e3%82%afdac%e6%a7%8b%e6%88%90/) | "conversor digital-analógico" | sigla é o termo de compra e de review; a forma longa não cabe em rótulo |
| `loudness_target_value`, `loudness_lufs` | %1$s LUFS | %1$s LUFS | jargão (unidade) | [Mazufa — Loudness no streaming: LUFS e true peak](https://www.distribution.mazufa.com/pt/loudness-reference/) | "LUFS (unidade de volume)" | unidade não se traduz; o artigo brasileiro usa LUFS cru |
| `loudness_normalization` | Loudness normalization | Normalização de volume | interface | [Mazufa](https://www.distribution.mazufa.com/pt/loudness-reference/) | "Normalização de loudness" | o ajuste é percebido como volume; "loudness" cru na interface pesa |
| `pipeline_loudness_gain` | Gain | Ganho | jargão (masculino) | uso corrente em mesas e plugins pt-BR | "Ganho de loudness" | rótulo curto de coluna; "ganho" é masculino, não "a ganho" |
| `cast`, `cast_casting` | Cast / Casting | Transmitir / Transmitindo | interface | [AOSP SystemUI pt-BR](https://android.googlesource.com/platform/frameworks/base/+/3860bee3d04acae3ff30c1d24c5f265ec5009a42/packages/SystemUI/res/values-pt-rBR/strings.xml); [MediaRouter pt-BR](https://android.googlesource.com/platform/frameworks/support/+/8394703ffbfb1ab86b1d23f3ca917a0754dc7e5c/mediarouter/mediarouter/src/main/res/values-pt-rBR/strings.xml) | "Cast" | é o vocabulário oficial do Android pt-BR para o botão e o estado |
| `cast_row_subtitle` | Chromecast and Google Cast speakers | Alto-falantes Chromecast e Google Cast | nome | idem | "caixas de som" | marca e protocolo ficam; "alto-falantes" é o termo do Android pt-BR |
| `bit_depth`, `pipeline_bit_depth` | %1$d-bit / Bit Depth | %1$d bits / Profundidade de bits | jargão + unidade | padrão brasileiro de escrita de unidades | "%1$d-bit", "Profundidade de bit" | sem hífen e no plural, como se escreve em pt-BR |
| `channel_count`, `pipeline_channels` | %1$d ch / Channels | %1$d canais / Canais | jargão + unidade | — | "%1$d ch" | "ch" é abreviação inglesa; "canais" é inequívoco |
| `mix_view_bars`, `mix_view_one_bar` | %1$d Bars / 1 Bar | %1$d compassos / 1 compasso | jargão de DJ | vocabulário de DJ/produção em pt-BR | "barras" | "bar" aqui é compasso musical; "barras" sugere gráfico |
| `mix_view_bpm` | %1$d bpm | %1$d BPM | jargão (unidade) | uso corrente em apps de DJ | "%1$d bpm" minúsculo | BPM é sigla em maiúsculas no Brasil |
| `autoplay*` | AutoPlay (on/off) | Reprodução automática (ativada/desativada) | interface | vocabulário de player Android pt-BR | "AutoPlay" | não é marca nem selo; o usuário lê a função |
| `mixing` | Mixing | Mixando | jargão | uso de DJ em pt-BR | "Misturando" | "mixar" é o verbo usado no meio; "misturar" descreve outra coisa na interface |
| `scrobbling`, `scrobbling_footer` | Scrobbling | Scrobbling | jargão | termo de Last.fm/ListenBrainz sem tradução consolidada | "registro de escuta" | quem usa Last.fm reconhece a palavra; a forma longa some no meio de um ajuste |
| `rich_presence` | Rich presence | Rich presence | nome de recurso (Discord) | nomenclatura do Discord | "presença detalhada" | é o nome do recurso no Discord |
| `listen_together*` | Listen together / party | Ouvir juntos / sessão | nome do produto + interface | rótulo do próprio recurso | "Ouvir junto", "festa" | "festa" não descreve sessão de escuta; "sessão" cabe em contador e em aviso |
| `downloaded`, `download_all`, `download_notification_title` | Downloaded / Download all / Downloading | Baixado / Baixar tudo / Baixando | interface | vocabulário Android pt-BR | "Downloaded", "Descarregar" | "baixar" é o verbo do Android pt-BR para download |
| `downloads`, `download_channel_name` | Downloads | Downloads | interface (nome de seção) | idem | "Baixados" | é o nome da seção/pasta, não uma ação; "Downloads" é o rótulo consagrado |
| `remove_from_liked`, `auto_liked` | Liked Music / Liked songs | Músicas curtidas | interface | vocabulário do YouTube Music pt-BR | "Músicas favoritas" | "curtir" é a ação do produto; "favoritas" conflita com "favoritas esquecidas" |
| `on_device`, `local_music` | On Device / Local Music | No aparelho / Música local | interface | vocabulário Android pt-BR ("aparelho" para device) | "No dispositivo" | "dispositivo" é mais longo e menos natural na UI |
| `smb`, `webdav` | SMB / WebDAV | SMB / WebDAV | nome de protocolo | — | — | protocolo não se traduz |
| `equalizer`, `system_equalizer` | Equalizer / System equalizer | Equalizador / Equalizador do sistema | interface | vocabulário Android pt-BR | "EQ" | "EQ" só onde o EN já abrevia (`pipeline_eq_preset`) |

Termo de jargão sem evidência citada volta para revisão.
