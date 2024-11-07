  # Angelica---CLOUD-COMPUTING-E-SRE
ATIVIDADE COMPLEMENTAR - ANGÉLICA DOS SANTOS RA: 10444130 

2.1 Timeout - 
2.1.2 Desafio Timeout: 
Ajustar configurações de timeout e corrigir erro de timeout execedido ao invocar o serviço

// INSIRA SUA ANÁLISE OU PARECER ABAIXO
// A - Diminuir o tempo de execução da função externalService para evitar que ela exceda o timeout definido.
// B - Ajustar o tempo limite (timeout) na função timeoutPromise ou no serviço externo.

      const express = require('express');
      const app = express();
      const port = 8080;
      // Função para criar uma Promise que simula um timeout
      function timeoutPromise(ms, promise) {
      return new Promise((resolve, reject) => {
        const timeout = setTimeout(() => {
            reject(new Error('Tempo limite excedido!'));
        }, ms);

        promise
            .then((result) => {
                clearTimeout(timeout);
                resolve(result);
            })
            .catch((error) => {
                clearTimeout(timeout);
                reject(error);
            });
    });
    }

    // Função simulando chamada externa
    async function externalService() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve('Resposta da chamada externa');
        }, 2000); // Ajustar o tempo para ser menor que o timeout
    });
    }

    // Rota de health check
    app.get('/api/health', (req, res) => {
    res.send('OK');
    });

    // Rota que faz a chamada simulada com timeout
    app.get('/api/timeout', async (req, res) => {
    try {
        const result = await timeoutPromise(3000, externalService());
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
    });

      // Iniciando o servidor
      app.listen(port, () => {
    console.log(`Servidor rodando em http://localhost:${port}`);
    });


      Executar a aplicação:
      bash
      node server-timeout.js

      Chamar o Endpoint:
      bash
      curl localhost:8080/api/timeout

Com essas alterações, a função externalService agora finaliza dentro do tempo limite de 3000 ms, evitando o erro de "tempo limite excedido".

2.2 Rate Limit - 
2.1.2 Desafio - Rate Limit : 
Alterar limite de requisições permitidas para 100 num intervalo de 1 minuto e escrever uma função para simular o erro de Rate Limit. 

// INSIRA SUA ANÁLISE OU PARECER ABAIXO
// A - Ajustar o limite de requisições para 100 por minuto.
// B - Escrever uma função para simular o erro de Rate Limit.

    const express = require('express');
    const rateLimit = require('express-rate-limit');

    const app = express();
    const port = 8080;

    // Middleware de rate limiting (Limite de 100 requisições por minuto)
    const limiter = rateLimit({
    windowMs: 60 * 1000,  // 1 minuto
    max: 100,  // Limite de 100 requisições
    message: 'Você excedeu o limite de requisições, tente novamente mais tarde.',
    });

    // Aplica o rate limiter para todas as rotas
    app.use(limiter);

    // Função simulando chamada externa
    async function externalService() {
    return 'Resposta da chamada externa';
    }

    // Rota que faz a chamada simulada
    app.get('/api/ratelimit', async (req, res) => {
    try {
        const result = await externalService();
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
    });

     // Função para simular erro de Rate Limit
     async function simulateRateLimit() {
    for (let i = 0; i < 110; i++) {  // Faz 110 requisições para garantir que exceda o limite
        await new Promise((resolve) => {
            const options = {
                hostname: 'localhost',
                port: 8080,
                path: '/api/ratelimit',
                method: 'GET',
            };

            const req = require('http').request(options, (res) => {
                let data = '';
                res.on('data', (chunk) => {
                    data += chunk;
                });
                res.on('end', () => {
                    console.log(`Requisição ${i+1}: ${data}`);
                    resolve();
                });
            });

            req.on('error', (e) => {
                console.error(`Problema com a requisição: ${e.message}`);
                resolve();
            });

            req.end();
        });
        }
        }

       // Iniciando o servidor
       app.listen(port, () => {
       console.log(`Servidor rodando em http://localhost:${port}`);
       simulateRateLimit();  // Simula as requisições ao iniciar o servidor
       });

      Executar a aplicação:
      bash
      node server-timeout.js

      Chamar o Endpoint:
      bash
      curl localhost:8080/api/timeout

Neste exemplo, a função simulateRateLimit faz 110 requisições para garantir que o limite de 100 requisições por minuto seja excedido, desencadeando o erro de Rate Limit.


2.3.2 Desafio - Bulkhead - 
Aumentar quantidade de chamadas simultâneas e avaliar o comportamento.

// INSIRA SUA ANÁLISE OU PARECER ABAIXO


       const express = require('express');
       const { bulkhead } = require('cockatiel');

    const app = express();
    const port = 8080;

    // Configurando bulkhead com cockatiel (Máximo de 10 requisições simultâneas)
    const bulkheadPolicy = bulkhead(10);

    // Função simulando chamada externa
    async function externalService() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve('Resposta da chamada externa');
        }, 2000);  // Simula uma chamada que demora 2 segundos
    });
    }

    // Rota que faz a chamada simulada
    app.get('/api/bulkhead', async (req, res) => {
    try {
        const result = await bulkheadPolicy.execute(() => externalService());
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
    });

    // Iniciando o servidor
    app.listen(port, () => {
    console.log(`Servidor rodando em http://localhost:${port}`);
     });

Com essas alterações, o servidor agora permitirá até 10 requisições simultâneas. Esse ajuste ajuda a garantir que a aplicação possa lidar com um maior número de requisições sem sobrecarregar o serviço.

BÔNUS: implementar método que utilizando threads para realizar as chamadas e logar na tela

// INSIRA SUA ANÁLISE OU PARECER ABAIXO

    const express = require('express');
    const { bulkhead } = require('cockatiel');
    const { Worker, isMainThread, parentPort } = require('worker_threads');

    const app = express();
    const port = 8080;

    // Configurando bulkhead com cockatiel (Máximo de 10 requisições simultâneas)
    const bulkheadPolicy = bulkhead(10);

     // Função simulando chamada externa
    async function externalService() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve('Resposta da chamada externa');
        }, 2000);  // Simula uma chamada que demora 2 segundos
    });
    }

     // Rota que faz a chamada simulada
     app.get('/api/bulkhead', async (req, res) => {
    try {
        const result = await bulkheadPolicy.execute(() => externalService());
        res.send(result);
    } catch (error) {
        res.status(500).send(`Erro: ${error.message}`);
    }
    });

    // Iniciando o servidor
     app.listen(port, () => {
    console.log(`Servidor rodando em http://localhost:${port}`);
     });

     // Se não for o thread principal, execute o worker
     if (!isMainThread) {
     parentPort.on('message', async () => {
        try {
            const result = await externalService();
            parentPort.postMessage(result);
        } catch (error) {
            parentPort.postMessage(`Erro: ${error.message}`);
        }
    });
    }
  
     // Função para executar múltiplas chamadas usando threads
     async function simulateBulkheadWithThreads() {
    const numThreads = 12;  // Número de threads para simular chamadas concorrentes

    for (let i = 0; i < numThreads; i++) {
        const worker = new Worker(__filename);
        worker.on('message', (message) => {
            console.log(`Thread ${i + 1}: ${message}`);
        });
        worker.postMessage('start');
    }
    }
       // Iniciar a simulação ao iniciar o servidor
       if (isMainThread) {
        simulateBulkheadWithThreads();
         }

Com essas alterações, você agora tem um exemplo que usa threads para simular chamadas concorrentes e loga os resultados na tela. Isso ajuda a avaliar o comportamento do bulkhead policy com múltiplas chamadas simultâneas.


2.4.1 Desafio - Circuit Breaker
Ajustar o o percentual de falhas para que o circuit breaker obtenha sucesso ao receber as requisições após sua abertura. Observar comportamento do circuito no console.

// INSIRA SUA ANÁLISE OU PARECER ABAIXO

      const express = require('express');
      const CircuitBreaker = require('opossum');

      const app = express();
      const port = 8080;

      // Função simulando chamada externa com falhas controladas
      async function externalService() {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
            const shouldFail = Math.random() > 0.5;  // Ajuste da taxa de falhas para 50%
            if (shouldFail) {
                reject(new Error('Falha na chamada externa'));
            } else {
                resolve('Resposta da chamada externa');
            }
        }, 2000);  // Simula uma chamada que demora 2 segundos
        });
      }

      // Configuração do Circuit Breaker
      const breaker = new CircuitBreaker(externalService, {
        timeout: 3000,  // Tempo limite de 3 segundos para a chamada
        errorThresholdPercentage: 50,  // Abre o circuito se 50% das requisições falharem
        resetTimeout: 10000  // Tenta fechar o circuito após 10 segundos
        });

      // Lidando com sucesso e falhas do Circuit Breaker
      breaker.fallback(() => 'Resposta do fallback...');
      breaker.on('open', () => console.log('Circuito aberto!'));
      breaker.on('halfOpen', () => console.log('Circuito meio aberto, testando...'));
      breaker.on('close', () => console.log('Circuito fechado novamente'));
      breaker.on('reject', () => console.log('Requisição rejeitada pelo Circuit Breaker'));
      breaker.on('failure', () => console.log('Falha registrada pelo Circuit Breaker'));
      breaker.on('success', () => console.log('Sucesso registrado pelo Circuit Breaker'));

      // Rota que faz a chamada simulada com o Circuit Breaker
      app.get('/api/circuitbreaker', async (req, res) => {
        try {
            const result = await breaker.fire();
            res.send(result);
        } catch (error) {
            res.status(500).send(`Erro: ${error.message}`);
        }
      });

      // Iniciando o servidor
      app.listen(port, () => {
          console.log(`Servidor rodando em http://localhost:${port}`);
          });

Quando o percentual de falhas atingir 50%, o circuito será aberto, rejeitando novas requisições até que o tempo de reset passe. Durante o estado meio aberto, ele testará as chamadas para decidir se fecha ou não o circuito novamente.





