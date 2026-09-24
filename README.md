# AMPERE — Monitor EMG

Painel web para o sistema AMPERE, que monitora o sinal EMG (eletromiografia) em um ESP32 e aciona uma EENM (eletroestimulação neuromuscular) quando detecta fadiga muscular.

Este repositório contém o **MVP publicável no GitHub Pages** (`index.html`), que reproduz o painel do sketch `AMPERE.ino` e permite testar com ou sem o hardware.

## Como funciona o sistema

O ESP32 lê o sensor EMG no pino 34, calcula o RMS de 100 amostras e o normaliza em % (0 a 100). A partir disso, uma máquina de estados decide quando ativar a EENM (o LED do pino 25 representa os pulsos):

| Estado | Fase | O que acontece |
|:---:|---|---|
| 0 | AGUARD | Monitoramento normal. Se o RMS ficar abaixo do limiar, vai para o estado 1. |
| 1 | CONF 1s | Confirma a fadiga: RMS abaixo do limiar por 1 s contínuo. Se voltar a subir, retorna ao estado 0. |
| 2 | ESPERA 5s | Espera mais 5 s com RMS abaixo do limiar. Se voltar a subir, retorna ao estado 0. |
| 3 | PULSOS | EENM ativa, com pulsos a ~30 Hz (16 ms ligado / 16 ms desligado). Desativa se ocorrerem 3 picos acima do limiar em menos de 5 s. |

Parâmetros usados (idênticos ao sketch):

| Parâmetro | Valor |
|---|---|
| Limiar de fadiga | 50 % do RMS máximo |
| Confirmação | 1000 ms |
| Espera para ativação | 5000 ms |
| Janela de picos / nº de picos | 5000 ms / 3 |
| Tensão máxima do sensor EMG | 2,0 V (referência do ADC: 3,3 V) |

## Usando o painel

### Modo simulação (padrão)

Não precisa de hardware. O controle deslizante **Esforço muscular simulado** define o RMS:

- Deixe em **80%** para operação normal (estado AGUARD).
- Clique em **FADIGA (20%)** e observe: CONF 1s → ESPERA 5s → PULSOS.
- Com a EENM ativa, clique em **PICO (80%)** e volte a **FADIGA** três vezes em menos de 5 s para ver a desativação.

### Modo Arduino real

1. Abra o `AMPERE.ino` na **Arduino IDE** (placa ESP32).
2. Preencha **Nome da rede** e **Senha** no painel e clique em **GERAR TRECHO PARA O SKETCH**.
3. Cole as duas linhas geradas (`ssid` e `password`) no topo do `.ino`.
4. Adicione no `setup()`, logo antes de `server.begin();`:

   ```cpp
   server.enableCORS(true);
   ```

5. Grave o sketch no ESP32 e abra o **Monitor Serial** (115200 baud) para ver o IP, por exemplo `192.168.0.50`.
6. No painel, digite o IP e clique em **CONECTAR AO ARDUINO**.

> **Por que a página não conecta o ESP32 ao Wi-Fi sozinha?** Navegadores não têm acesso para configurar a rede de um dispositivo externo. Por isso o SSID e a senha servem para gerar o trecho do sketch. Nada é enviado a servidores, e a senha fica apenas na aba aberta.

## Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| Status "sem resposta" | Dispositivo e ESP32 em redes diferentes, ou IP errado | Conferir a rede e o IP no Monitor Serial |
| Requisição bloqueada no console | O GitHub Pages usa HTTPS e o ESP32 responde em HTTP (conteúdo misto) | No Chrome/Edge: ícone de cadeado → Configurações do site → **Conteúdo não seguro → Permitir** |
| Erro de CORS | Faltou `server.enableCORS(true);` no sketch | Adicionar a linha e regravar |
| Ainda bloqueado pelo Chrome | Restrições de acesso à rede local | Abrir o `index.html` direto do computador, ou usar o endereço `http://IP` do próprio ESP32, que serve a página original |

## Estrutura

```
.
├── index.html   # MVP para o GitHub Pages
└── README.md
```

O sketch original (`AMPERE.ino`) roda na Arduino IDE e não precisa ser publicado no Pages.

## Dependências

- [Chart.js 4.4.1](https://www.chartjs.org/) (CDN jsDelivr)
- Fontes Share Tech Mono e Rajdhani (Google Fonts)

## Aviso

Projeto de pesquisa/prototipagem. Não é um dispositivo médico validado e não deve ser usado para diagnóstico ou tratamento sem supervisão profissional.
