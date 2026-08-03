# Documentação técnica — LIGHT Login Experience

## 1. Visão geral

O LIGHT é uma aplicação front-end de página única construída com Next.js e
React. Seu objetivo é demonstrar como iluminação, movimento, som e resposta de
interface podem transformar um formulário comum em uma experiência interativa.

A aplicação é exportada como conteúdo estático. Não existe servidor próprio,
banco de dados ou serviço de autenticação nesta versão.

## 2. Escopo funcional

O projeto implementa:

- cena visual responsiva com abajur;
- controle de luz pelo cordão;
- clique, toque, arraste e teclado;
- movimento pendular do cordão;
- síntese sonora para ações da interface;
- formulário demonstrativo com validação do navegador;
- estado visual de acesso realizado;
- metadados para compartilhamento social;
- build estático preparado para GitHub Pages.

Não fazem parte do escopo atual:

- autenticação real;
- criação ou consulta de usuários;
- armazenamento de credenciais;
- recuperação de senha;
- sessão persistente;
- área privada após o login.

## 3. Arquitetura

### 3.1 Camadas principais

| Camada | Arquivo | Responsabilidade |
| --- | --- | --- |
| Documento e metadados | `app/layout.tsx` | Idioma, título, descrição, Open Graph e Twitter Card. |
| Interface e comportamento | `app/page.tsx` | Estados React, eventos de ponteiro, validação e síntese sonora. |
| Apresentação | `app/globals.css` | Layout, iluminação, cordão, formulário, animações e responsividade. |
| Build | `next.config.ts` | Exportação estática, `basePath`, prefixo de assets e imagens sem otimização no servidor. |
| Entrega | `.github/workflows/deploy-pages.yml` | Instalação, build, empacotamento e publicação no GitHub Pages. |

### 3.2 Estados da interface

| Estado | Tipo | Uso |
| --- | --- | --- |
| `lightOn` | `boolean` | Controla iluminação, visibilidade e disponibilidade do formulário. |
| `pulling` | `boolean` | Ativa a animação curta de puxar o cordão. |
| `dragging` | `boolean` | Indica interação contínua por ponteiro ou toque. |
| `chainAngle` | `number` | Define o ângulo atual do pêndulo em graus. |
| `chainLength` | `number` | Define o comprimento visual do cordão em porcentagem. |
| `message` | `string` | Exibe a confirmação local do formulário. |
| `authenticated` | `boolean` | Ativa o indicador visual de sucesso demonstrativo. |

Referências mutáveis armazenam o contexto de áudio, a origem do arraste e os
limites de repetição sonora sem provocar renderizações desnecessárias.

## 4. Interação do cordão

O botão `.pull-switch` é a área interativa do cordão. Durante o arraste, a
posição do ponteiro é comparada ao pivô da luminária. O deslocamento horizontal
é limitado e convertido em ângulo com `Math.asin`, produzindo um movimento
pendular coerente.

O deslocamento vertical acrescenta comprimento temporário ao cordão. As
variáveis CSS `--chain-angle` e `--chain-length` recebem os valores calculados e
atualizam o elemento visual sem recriar sua estrutura.

Uma tolerância de quatro pixels diferencia clique de arraste. Isso evita que a
luz seja alternada acidentalmente após o usuário movimentar o cordão.

## 5. Sistema de áudio

Os sons são produzidos em tempo real pela Web Audio API. O projeto combina:

- `OscillatorNode` para tons e corpo sonoro;
- `AudioBufferSourceNode` para ruído sintético;
- `BiquadFilterNode` para selecionar frequências;
- `GainNode` para controlar ataque e decaimento.

Há quatro respostas principais:

| Evento | Resposta sonora |
| --- | --- |
| Movimento do cordão | Pequenos ruídos metálicos proporcionais à variação do ângulo. |
| Interruptor | Clique mecânico, corpo grave e, ao acender, breve energia elétrica. |
| Envio válido | Sequência ascendente de três notas e pulso grave. |
| Erro de validação | Tom descendente curto, com proteção contra repetição excessiva. |

O `AudioContext` só é criado após interação do usuário, respeitando as regras de
reprodução automática dos navegadores modernos.

## 6. Iluminação e animação

A classe `light-on` é aplicada ao elemento principal quando o abajur está
aceso. A partir dela, o CSS coordena:

- brilho e saturação da cena;
- opacidade da camada de escurecimento;
- entrada e saída do formulário;
- bloqueio ou liberação de eventos do formulário;
- tempo das transições.

O cordão usa `transform-origin` no ponto superior e uma curva de animação com
retorno elástico. Durante o arraste, as transições são removidas para manter a
resposta direta ao ponteiro.

## 7. Responsividade

Em telas horizontais, a cena mantém a proporção `1536 / 909` e fica centralizada
dentro da viewport. Esse comportamento preserva a posição relativa do abajur,
do cordão e do formulário.

Em telas com proporção inferior a `4 / 5`, a aplicação muda para o modo vertical:

- a cena ocupa a largura disponível;
- a arte é ampliada sem deformação;
- o enquadramento é deslocado para manter o abajur visível;
- o cordão passa a usar medidas relacionadas à largura da viewport;
- o formulário é reposicionado abaixo da área principal da luminária;
- a página permite rolagem vertical quando necessário.

## 8. Acessibilidade

Foram aplicados os seguintes cuidados:

- `aria-label` descritivo para acender ou apagar o abajur;
- `aria-pressed` para comunicar o estado do interruptor;
- foco visual para navegação por teclado;
- rótulos ocultos para leitores de tela;
- mensagem de resultado com `role="status"`;
- remoção do formulário da ordem de tabulação quando a luz está apagada;
- redução quase total das animações com `prefers-reduced-motion`.

## 9. Formulário e dados

O navegador valida:

- formato do e-mail por `type="email"`;
- preenchimento obrigatório por `required`;
- senha com no mínimo seis caracteres por `minLength={6}`.

O evento `handleSubmit` cancela o envio HTTP, altera o indicador para sucesso e
reproduz a confirmação sonora. Os valores digitados não saem do navegador e não
são salvos.

## 10. Metadados sociais

O `app/layout.tsx` centraliza título, descrição e imagens sociais. As variáveis
abaixo permitem gerar URLs corretas em ambiente local e no GitHub Pages:

| Variável | Exemplo | Finalidade |
| --- | --- | --- |
| `NEXT_PUBLIC_SITE_URL` | `https://xx7rg.github.io/LIGHT-Login-Experience` | Endereço público usado nos metadados. |
| `NEXT_PUBLIC_BASE_PATH` | `/LIGHT-Login-Experience` | Prefixo necessário quando o site não está na raiz do domínio. |

## 11. Build e publicação

`next.config.ts` utiliza `output: "export"`. Assim, `npm run build` gera a pasta
`out/` com HTML, JavaScript, CSS e assets prontos para hospedagem estática.

O workflow do GitHub Actions roda em cada push para `main`, configura o caminho
do repositório, executa o build e publica o artefato no GitHub Pages. A aplicação
não depende de funções server-side durante a execução.

## 12. Validação de qualidade

Antes de publicar uma alteração, execute:

```bash
npm ci
npm run lint
npm run build
```

O lint cobre regras de JavaScript, TypeScript, React e acessibilidade. O build
também executa a verificação de tipos e confirma que a exportação estática pode
ser produzida.

## 13. Guia de personalização

### Cores

As variáveis principais estão no início de `app/globals.css`:

```css
:root {
  --gold: #dca552;
  --ivory: #f1e7d1;
}
```

### Textos e autoria

Título, subtítulo, créditos e campos ficam em `app/page.tsx`.

### Imagem da cena

A imagem principal é `public/lamp-scene-v2.png`. Se for substituída, mantenha a
mesma proporção ou revise as posições percentuais do cordão e do formulário.

### Imagem social

A prévia usada por redes sociais é `public/og-v3.png` e está configurada em
`app/layout.tsx`.

## 14. Evolução para autenticação real

Para transformar a demonstração em produto, recomenda-se:

1. escolher um provedor de identidade confiável;
2. trocar o sucesso local por uma chamada segura ao servidor;
3. nunca registrar senhas em logs ou armazenamento do navegador;
4. usar sessão segura com expiração e renovação;
5. implementar limitação de tentativas e mensagens de erro neutras;
6. adicionar recuperação de senha e verificação de e-mail;
7. proteger rotas privadas no servidor;
8. criar testes funcionais e de acessibilidade.

## 15. Resumo técnico

O valor do LIGHT não está na quantidade de telas, mas na integração cuidadosa
entre arte, estado, movimento, áudio, responsividade e acessibilidade. A base é
pequena, estática e fácil de publicar, mas utiliza técnicas normalmente
encontradas em interfaces de produto mais elaboradas.
