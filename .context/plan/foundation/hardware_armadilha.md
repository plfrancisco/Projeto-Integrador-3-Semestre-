# HARDWARE DA ARMADILHA — StickFly K-45

**Versão:** 1.0
**Status:** ATIVO
**Escopo:** especificações do equipamento real e implicações para a visão
computacional

---

# 1. Objetivo

Registrar as especificações do equipamento da empresa parceira e derivar as
implicações técnicas para a geração do dataset sintético, o posicionamento da
câmera e o pré-processamento das imagens.

---

# 2. Especificações técnicas

| Item | Especificação |
|---|---|
| Modelo | StickFly K-45 |
| Dimensões (C × A × L) | 50 cm × 27 cm × 22 cm |
| Material do corpo | Aço com pintura epóxi branca / aço inox / ABS de alta resistência |
| Instalação | Parede |
| Lâmpadas | 3× 15 W BL UVA |
| **Refil adesivo** | **450 × 220 mm** |
| Voltagem | 110 V ou 220 V (chave seletora) |
| Consumo | 0,045 kW/h |
| Cobertura | 60 m² |
| Peso | 4,280 kg |

---

# 3. Implicações para a visão computacional

## 3.1 Superfície de interesse

O objeto a ser analisado é o **refil adesivo de 450 × 220 mm**, não a
armadilha inteira. A razão de aspecto é de aproximadamente **2,05:1**, o que
orienta:

- as dimensões das imagens sintéticas geradas;
- o crop ou retificação da imagem real antes de entrar no modelo;
- a resolução de entrada da rede — manter proporção próxima evita distorção.

## 3.2 Área física por pixel

Como as dimensões do refil são conhecidas, é possível converter percentual de
área coberta em **área física real** (cm² cobertos), caso isso seja útil para
o relatório ou para calibrar o limiar de troca.

## 3.3 Iluminação

As três lâmpadas BL UVA de 15 W ficam próximas à superfície adesiva. Isso
implica:

- iluminação artificial constante e dominante sobre a placa;
- possível dominância de tom violeta/azulado nas imagens, dependendo de como
  a câmera captura a luz UV;
- risco de reflexo ou brilho especular na superfície adesiva;
- iluminação relativamente controlada, o que é favorável — mas o modelo deve
  tolerar variação, já que o estabelecimento também tem luz própria.

Essas condições devem ser reproduzidas na geração do dataset sintético e na
augmentation.

## 3.4 Instalação em parede

A armadilha é fixa em parede, o que significa:

- ângulo de captura da câmera é fixo e previsível por instalação;
- a distorção de perspectiva tende a ser constante para uma mesma unidade;
- há variação entre instalações diferentes (altura, distância da câmera), o
  que a augmentation deve cobrir.

## 3.5 Cor de fundo

O corpo é branco (pintura epóxi) ou aço inox. A cor do refil adesivo ainda
precisa ser confirmada — tipicamente esses refis são amarelos, o que favorece
o contraste com os insetos (escuros) e beneficia tanto o baseline OpenCV
quanto o modelo treinado.

---

# 4. Pendências

- [ ] Confirmar a cor do refil adesivo
- [ ] Obter fotos reais do equipamento em uso, em diferentes níveis de
      saturação
- [ ] Definir posição e especificação da câmera (resolução, distância,
      ângulo)
- [ ] Verificar se o refil tem marcações, logotipo ou bordas impressas que
      possam ser confundidas com insetos pela segmentação

---

# FIM DO HARDWARE DA ARMADILHA
