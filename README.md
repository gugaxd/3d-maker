# 3d maker

Gerador de formas 3D para design gráfico e motion. Roda inteiramente no navegador, sem back-end.

## O que faz

- **Formas** — 12 primitivas paramétricas (cubo, esfera, cilindro, cone, toro, nó de toro, os cinco sólidos, anel, plano) mais importação de SVG próprio
- **SVG importado** — alternável entre extrudado, com profundidade e chanfro, e plano; contornos internos viram furos automaticamente
- **Cena** — um objeto ou vários, em lista, com seleção por clique no palco
- **Transformação** — posição, rotação e escala por eixo, escala uniforme e translucência
- **Material** — fosco, brilhante, metal, vidro, chapado e arame, com cor e rugosidade
- **Animação** — timeline com keyframes por objeto, interpolação linear ou suave, e atalho para giro de 360°
- **Exportação** — PNG com alpha, SVG vetorial e WebM com canal alpha

## Detalhes técnicos

A translucência não é opacidade linear: o alpha é modulado por Fresnel dentro do shader, via
`onBeforeCompile`. A silhueta, o chanfro e as curvas de fuga continuam densos e só o miolo deixa
passar, que é como vidro se comporta. No máximo o alpha de base para em 7%, então o objeto nunca
some. O `envMapIntensity` sobe junto, para o reflexo crescer com a translucência.

O SVG importado é convertido por um parser próprio de `path`, `rect`, `circle`, `ellipse`,
`polygon` e `polyline`, com suporte a `transform` acumulado. Curvas e arcos são amostrados em
polilinhas, o que torna a triangulação previsível. Contornos contidos dentro de outros viram
`holes` da `Shape`, detectados por área e ponto-em-polígono.

A exportação SVG não usa o `SVGRenderer`: cada triângulo é projetado pela câmera, sombreado por
Lambert com o mesmo Fresnel do viewport e ordenado por distância — algoritmo do pintor. Sai vetor
puro, face a face, pronto para abrir no Illustrator. Menos detalhe na forma significa arquivo
bem mais leve.

O vídeo sai por `MediaRecorder` sobre o `captureStream` do canvas, que preserva o canal alpha. O
After Effects não abre WebM direto; converta antes:

```bash
ffmpeg -i 3d-maker-alpha.webm -c:v prores_ks -profile:v 4444 -pix_fmt yuva444p10le saida.mov
```

A órbita da câmera é própria, em coordenadas esféricas, sem `OrbitControls`.

## Sistema visual

Paleta, tipografia, `Header` e `Footer` vêm do [gri.d.maker](https://github.com/gugaxd/gri.d.maker),
que é a fonte da verdade da família. Ao mudar qualquer token lá, replique aqui.

Uma pendência: no gri.d.maker a Host Grotesk SemiBold entra como subconjunto base64 com os glifos
de "gri.d.maker". O "3" não está nesse subconjunto, então aqui a fonte vem da Google Fonts. Gere o
subconjunto de "3d maker" e troque por `@font-face` embutido para cortar a chamada externa.

## Rodando localmente

```bash
npm install
npm run dev
```

## Publicando

Hospedado na Vercel. A Vercel observa a `main` deste repositório e publica sozinha a cada push —
não há workflow de deploy aqui. O build sai na raiz (`base` padrão), então não existe prefixo de
caminho a manter em sincronia.

O bundle passa de 500 kB porque carrega o three.js inteiro; o aviso do Vite no build é esperado.

## Licença

MIT
