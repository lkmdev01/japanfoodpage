# Kami — Sushi feito para sentir

Landing page em Vue 3 + TypeScript, Vite, Tailwind CSS 4 e GSAP/ScrollTrigger. Inspirada nas referências fornecidas, com vermelho principal `#FE000E`, tipografia editorial e ilustrações originais geradas para o projeto.

## Rodar

```sh
npm install
npm run dev
```

```sh
npm run build
npm run preview
```

## Estrutura

- `src/App.vue`: seções, cardápio, busca, sacola e animações.
- `src/style.css`: identidade visual, responsividade e movimento reduzido.
- `public/images/`: ilustrações WebP com transparência.

O cardápio e os preços são demonstrativos e podem ser alterados no array `dishes` em `App.vue`. A sacola permite adicionar, remover e totalizar itens em memória. Não há checkout, pagamento ou processamento de pedidos. O formulário valida o e-mail, mas informa que as inscrições ainda não estão disponíveis; não armazena nem envia dados. Conectar os serviços reais antes de publicar como restaurante comercial.

As fontes são carregadas pelo Google Fonts. Para hospedagem sem dependências externas, hospede os arquivos de fonte localmente e atualize o CSS. Animações respeitam `prefers-reduced-motion`; a sacola usa o elemento nativo `dialog` para foco e fechamento por Escape.

## Referência

Mantém a sequência: hero, apresentação, pratos populares, destaque, newsletter e rodapé. Os assets foram recriados e a marca e os textos adaptados para Kami; não é uma cópia pixel a pixel. Nenhum arquivo do projeto institucional InovaForce foi alterado.
