# 💳 Guia Completo de Pagamento - MáquinasMax

## 🎯 Visão Geral

O sistema de pagamento do MáquinasMax suporta 3 formas principais:
- ✅ **PIX** - Instantâneo
- ✅ **Cartão de Crédito** - Parcelado até 12x
- ✅ **Boleto** - Compensação em 2 dias

---

## 📁 Arquivos de Pagamento

```
js/
├── payment.js           ← Sistema de pagamento
├── main.js              ← Validações gerais

pages/checkout/
├── pagamento.html       ← Interface de checkout
└── locacoes.html        ← Carrinho/histórico
```

---

## 🔧 Como Funciona Atualmente (Simulado)

### 1️⃣ **PIX**
```javascript
// Usuário insere chave PIX
// Sistema gera QR Code (SVG simulado)
// Em produção: integrar com API do banco
```

### 2️⃣ **Cartão de Crédito**
```javascript
// Validação de:
// - Número do cartão (Luhn)
// - Data de validade
// - CVV
// - Dados do titular
// - CPF

// Em produção: enviar para gateway seguro
// Nunca armazenar dados de cartão no cliente
```

### 3️⃣ **Boleto**
```javascript
// Usuário insere CPF
// Sistema gera boleto (simulado)
// Em produção: integrar com banco
```

---

## 🚀 Integração com Gateways Reais

### Opção 1: **MercadoPago** (Recomendado Brasil)

**Vantagens:**
- Suporta PIX, cartão, boleto
- SDK pronto
- Documentação excelente
- Melhor para checkout

**Instalação:**

```html
<!-- 1. Adicionar script no HTML -->
<script src="https://sdk.mercadopago.com/js/v2"></script>

<!-- 2. Inicializar no JavaScript -->
<script>
const mp = new MercadoPago('PUBLIC_KEY');

// 3. Criar checkout
mp.checkout({
    preference: {
        id: 'PREFERENCE_ID' // Vem do seu backend
    },
    autoOpen: true
});
</script>
```

**Fluxo Backend:**

```python
# Python exemplo
import mercadopago
from flask import Flask, jsonify

sdk = mercadopago.SDK("ACCESS_TOKEN")

@app.route('/api/pagamento/preferencia', methods=['POST'])
def criar_preferencia():
    preference_data = {
        "items": [
            {
                "title": "Escavadeira CAT 320",
                "quantity": 1,
                "unit_price": 850000
            }
        ],
        "back_urls": {
            "success": "https://seu-site.com/sucesso",
            "failure": "https://seu-site.com/falha",
            "pending": "https://seu-site.com/pendente"
        },
        "auto_return": "approved"
    }

    preference_response = sdk.preference().create(preference_data)
    return jsonify(preference_response)
```

---

### Opção 2: **Stripe** (Global)

**Vantagens:**
- Suporte internacional
- PIX via Stripe
- Documentação completa

```javascript
// 1. Adicionar script
<script src="https://js.stripe.com/v3/"></script>

// 2. Inicializar
const stripe = Stripe('CHAVE_PUBLICA');
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element');

// 3. Processar pagamento
stripe.confirmCardPayment(clientSecret, {
    payment_method: {
        card: cardElement
    }
});
```

---

### Opção 3: **PagSeguro** (Brasil)

**Vantagens:**
- Processo simples
- Suporte local
- Boa taxa

```javascript
// Integração similar ao MercadoPago
PagSeguroDirectPayment.setSessionId(sessionId);

// Criar token de cartão
PagSeguroDirectPayment.createCardToken({
    cardNumber: "1234567812345678",
    cvv: "123",
    // ... mais dados
    onSuccess: function(response) {
        // Enviar token ao backend
    }
});
```

---

## 📋 Implementação Passo a Passo

### Passo 1: Configurar Variáveis de Ambiente

**.env**
```
MERCADOPAGO_PUBLIC_KEY=APP_123456
MERCADOPAGO_ACCESS_TOKEN=APP_ACCESS_TOKEN_123456
STRIPE_PUBLIC_KEY=pk_test_123456
STRIPE_SECRET_KEY=sk_test_123456
```

### Passo 2: Criar Serviço de Pagamento no Backend

**backend/services/pagamento.js**
```javascript
const mercadopago = require('mercadopago');

class PagamentoService {
    constructor() {
        mercadopago.configure({
            access_token: process.env.MERCADOPAGO_ACCESS_TOKEN
        });
    }

    async criarPreferencia(itens, usuarioId) {
        try {
            const preference = {
                items: itens.map(item => ({
                    title: item.nome,
                    quantity: item.quantidade,
                    unit_price: parseFloat(item.preco)
                })),
                payer: {
                    email: `usuario${usuarioId}@maquinasmax.com`
                },
                back_urls: {
                    success: `${process.env.FRONTEND_URL}/sucesso`,
                    failure: `${process.env.FRONTEND_URL}/falha`,
                    pending: `${process.env.FRONTEND_URL}/pendente`
                },
                auto_return: "approved"
            };

            const response = await mercadopago.preferences.create(preference);
            return response.body.init_point; // URL para redirecionar
        } catch (error) {
            console.error('Erro ao criar preferência:', error);
            throw error;
        }
    }

    async validarPagamento(paymentId) {
        try {
            const response = await mercadopago.payment.get(paymentId);
            return response.body;
        } catch (error) {
            console.error('Erro ao validar pagamento:', error);
            throw error;
        }
    }
}

module.exports = PagamentoService;
```

### Passo 3: Criar Endpoints da API

**backend/routes/pagamento.js**
```javascript
const express = require('express');
const router = express.Router();
const PagamentoService = require('../services/pagamento');

const pagamento = new PagamentoService();

// POST /api/pagamento/preferencia
router.post('/preferencia', async (req, res) => {
    try {
        const { itens, usuarioId } = req.body;
        const checkoutUrl = await pagamento.criarPreferencia(itens, usuarioId);
        
        res.json({
            success: true,
            checkout_url: checkoutUrl
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
});

// POST /api/pagamento/validar
router.post('/validar/:paymentId', async (req, res) => {
    try {
        const { paymentId } = req.params;
        const pagamentoData = await pagamento.validarPagamento(paymentId);
        
        res.json({
            success: true,
            status: pagamentoData.status,
            data: pagamentoData
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: error.message
        });
    }
});

// Webhook (validar pagamento quando vem de fora)
router.post('/webhook', (req, res) => {
    const { data, type } = req.body;
    
    if (type === 'payment') {
        // Atualizar status no banco de dados
        console.log('Pagamento recebido:', data);
    }
    
    res.json({ received: true });
});

module.exports = router;
```

### Passo 4: Atualizar Frontend

**js/payment.js - Adicionar**
```javascript
async enviarPagamento(dados) {
    try {
        const response = await fetch('/api/pagamento/preferencia', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('token')}`
            },
            body: JSON.stringify(dados)
        });

        const result = await response.json();
        
        if (result.success) {
            // Redirecionar para checkout do MercadoPago
            window.location.href = result.checkout_url;
        } else {
            throw new Error(result.error);
        }
    } catch (error) {
        alert('Erro ao processar pagamento: ' + error.message);
    }
}
```

---

## 🔒 Segurança

### ⚠️ **NUNCA** fazer:
```javascript
// ❌ ERRADO - Nunca armazenar ou enviar números de cartão completos
const dados = {
    numeroCartao: '1234567890123456',
    cvv: '123'
};
fetch('/api/pagamento', { body: JSON.stringify(dados) });
```

### ✅ **SEMPRE** fazer:
```javascript
// ✅ CORRETO - Usar tokenização
stripe.createToken(cardElement).then(result => {
    // Usar token, não dados reais
    fetch('/api/pagamento', {
        body: JSON.stringify({
            stripeToken: result.token.id // Token, não dados sensíveis
        })
    });
});
```

### Checklist de Segurança
- [ ] Nunca armazenar dados de cartão no cliente
- [ ] Usar HTTPS em produção
- [ ] Tokenizar dados sensíveis
- [ ] Validar no backend também
- [ ] Usar 3D Secure para cartão
- [ ] Implementar PCI DSS
- [ ] Rate limiting em endpoints
- [ ] Logging de transações
- [ ] Alertas de fraude

---

## 📊 Fluxo Completo de Pagamento

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Usuário no Carrinho                                      │
│    └─> Clica "Ir para Pagamento"                            │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Página de Pagamento (pagamento.html)                     │
│    ├─> PIX: Input chave PIX + QR Code                       │
│    ├─> Cartão: Formulário com validação                     │
│    └─> Boleto: Input CPF                                    │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Submeter Formulário (payment.js)                         │
│    ├─> Validar dados locais                                 │
│    ├─> Tokenizar (se cartão)                                │
│    └─> Enviar ao Backend                                    │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Backend Processa                                         │
│    ├─> Validar dados                                        │
│    ├─> Chamar Gateway de Pagamento                          │
│    ├─> Salvar transação no BD                               │
│    └─> Retornar resultado                                   │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Gateway Processa (MercadoPago/Stripe/etc)               │
│    ├─> Validar pagamento                                    │
│    ├─> Debitar valor                                        │
│    └─> Enviar notificação (webhook)                         │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Webhook Backend Recebe                                   │
│    ├─> Confirmar pagamento                                  │
│    ├─> Atualizar status no BD                               │
│    ├─> Enviar confirmação por email                         │
│    └─> Liberar acesso ao cliente                            │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. Página de Sucesso                                        │
│    └─> Mostrar confirmação ao usuário                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📞 Testes

### Teste com Dados Reais (Sandbox)

**MercadoPago - Cartões de Teste:**
```
Visa:        4111111111111111
MasterCard:  5555555555554444
CVV:         123 (qualquer)
Data:        01/25 (futura)
```

**Stripe - Cartões de Teste:**
```
Visa:        4242 4242 4242 4242
MasterCard:  5555 5555 5555 4444
CVV:         123 (qualquer)
```

### Teste PIX Simulado
```
Chave: seu-email@exemplo.com
ou
Chave: 12345678900 (CPF)
ou
Chave: 11999999999 (Telefone)
```

---

## 🚨 Tratamento de Erros

```javascript
// Possíveis erros
const erros = {
    'INSUFFICIENT_FUNDS': 'Saldo insuficiente',
    'CARD_DECLINED': 'Cartão recusado',
    'EXPIRED_CARD': 'Cartão expirado',
    'INVALID_CVV': 'CVV inválido',
    'PROCESSING_ERROR': 'Erro ao processar',
    'NETWORK_ERROR': 'Erro de conexão'
};
```

---

## 📈 Próximas Melhorias

- [ ] Suporte a múltiplas moedas
- [ ] Recorrência/Assinatura
- [ ] Parcelamento automático
- [ ] Antifraude integrado
- [ ] Dashboard de transações
- [ ] Relatórios fiscal
- [ ] Integração contábil
- [ ] Reembolso automatizado

---

**🎉 Seu sistema de pagamento está pronto!**

Para dúvidas, consulte:
- MercadoPago: https://www.mercadopago.com.br/developers
- Stripe: https://stripe.com/docs
- PagSeguro: https://pagseguro.uol.com.br/dev
