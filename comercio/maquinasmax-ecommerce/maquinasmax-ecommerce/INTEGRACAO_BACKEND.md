# 🔌 Integração Backend - MáquinasMax

Guia para integrar o frontend com sua API backend.

## 📡 Endpoints Necessários

### Autenticação

#### POST `/api/auth/cadastro`
Cria nova conta de usuário
```javascript
const dados = {
    nome: string,
    cpf: string,
    email: string,
    senha: string (criptografada no backend),
    telefone: string,
    cidade: string,
    estado: string,
    cep: string,
    endereco: string
};
```

#### POST `/api/auth/login`
Autentica usuário
```javascript
const dados = {
    email: string,
    senha: string
};

// Response
{
    token: "jwt_token",
    usuario: { id, nome, email }
}
```

#### POST `/api/auth/recuperar-senha`
Solicita reset de senha
```javascript
const dados = {
    email: string
};
```

#### POST `/api/auth/resetar-senha`
Reseta senha com token
```javascript
const dados = {
    token: string,
    nova_senha: string
};
```

---

### Produtos/Anúncios

#### GET `/api/produtos`
Lista todos os produtos com filtros
```javascript
// Query params
?categoria=caminhao
?tipo=aluguel
?busca=escavadeira
?pagina=1
?limite=20
```

#### GET `/api/produtos/:id`
Obtém detalhes de um produto
```javascript
// Response
{
    id: number,
    nome: string,
    descricao: string,
    preco: number,
    tipo: "venda" | "aluguel",
    categoria: string,
    imagem: string,
    vendedor: { id, nome, email, rating },
    estrelas: number,
    criado_em: date
}
```

#### POST `/api/anuncios` (requer auth)
Cria novo anúncio
```javascript
const dados = new FormData();
dados.append('nome', string);
dados.append('descricao', string);
dados.append('preco', number);
dados.append('tipo', "venda" | "aluguel");
dados.append('categoria', string);
dados.append('imagem', File);
dados.append('disponibilidade', string);
```

#### GET `/api/meus-anuncios` (requer auth)
Lista anúncios do usuário logado

#### DELETE `/api/anuncios/:id` (requer auth)
Remove um anúncio

---

### Chat

#### POST `/api/chat/enviar` (requer auth)
Envia mensagem
```javascript
const dados = {
    receptor_id: number,
    mensagem: string,
    anuncio_id: number (opcional)
};
```

#### GET `/api/chat/mensagens/:usuario_id` (requer auth)
Obtém histórico de chat

---

### Transações

#### POST `/api/transacoes/criar` (requer auth)
Cria nova transação
```javascript
const dados = {
    anuncio_id: number,
    quantidade: number,
    data_inicio: date,
    data_fim: date,
    valor_total: number,
    metodo_pagamento: "PIX" | "BOLETO" | "CARTAO"
};
```

#### GET `/api/minhas-transacoes` (requer auth)
Lista transações do usuário

#### GET `/api/transacoes/:id` (requer auth)
Obtém detalhe de uma transação

---

## 🔐 Autenticação JWT

### Implementar no Frontend

```javascript
// Salvar token após login
const token = response.data.token;
localStorage.setItem('token', token);

// Enviar token em requisições autenticadas
async function requisiçãoAutenticada(endpoint, dados) {
    const token = localStorage.getItem('token');
    
    const response = await fetch(`http://seu-backend.com${endpoint}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(dados)
    });
    
    return response.json();
}

// Remover token ao fazer logout
localStorage.removeItem('token');
```

### Verificar Token Expirado

```javascript
function tokenExpirado() {
    const token = localStorage.getItem('token');
    if (!token) return true;
    
    try {
        const payload = JSON.parse(atob(token.split('.')[1]));
        return payload.exp * 1000 < Date.now();
    } catch {
        return true;
    }
}

if (tokenExpirado()) {
    window.location.href = '/pages/auth/login.html';
}
```

---

## 📧 Email

### Verificação de Email

Após cadastro, enviar email com link:
```
https://seu-dominio.com/pages/auth/verificar-email.html?token=ABC123
```

### Recuperação de Senha

Enviar email com link:
```
https://seu-dominio.com/pages/auth/resetar-senha.html?token=XYZ789
```

---

## 💳 Pagamentos

Integrar com:
- **MercadoPago** - Recomendado para Brasil
- **Stripe** - Suporta PIX
- **PayPal** - Alternativa

### Fluxo

1. Frontend envia dados para backend
2. Backend cria "preference" no gateway
3. Backend retorna redirect_url
4. Frontend redireciona usuário
5. Gateway processa pagamento
6. Gateway retorna para seu URL de sucesso
7. Backend valida e atualiza BD

---

## 🗄️ Estrutura de Banco de Dados (Recomendado)

```sql
-- Usuários
CREATE TABLE usuarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(255),
    cpf VARCHAR(14) UNIQUE,
    email VARCHAR(255) UNIQUE,
    senha VARCHAR(255),
    telefone VARCHAR(20),
    cidade VARCHAR(100),
    estado VARCHAR(2),
    cep VARCHAR(10),
    endereco TEXT,
    rating DECIMAL(3,1),
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    verificado INT DEFAULT 0
);

-- Produtos/Anúncios
CREATE TABLE anuncios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT,
    nome VARCHAR(255),
    descricao TEXT,
    preco DECIMAL(12,2),
    tipo ENUM('venda', 'aluguel'),
    categoria VARCHAR(50),
    imagem VARCHAR(500),
    disponivel INT DEFAULT 1,
    visualizacoes INT DEFAULT 0,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

-- Transações
CREATE TABLE transacoes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    anuncio_id INT,
    comprador_id INT,
    vendedor_id INT,
    valor DECIMAL(12,2),
    metodo_pagamento VARCHAR(20),
    status ENUM('pendente', 'confirmado', 'cancelado'),
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (anuncio_id) REFERENCES anuncios(id),
    FOREIGN KEY (comprador_id) REFERENCES usuarios(id),
    FOREIGN KEY (vendedor_id) REFERENCES usuarios(id)
);

-- Chat
CREATE TABLE mensagens (
    id INT PRIMARY KEY AUTO_INCREMENT,
    remetente_id INT,
    receptor_id INT,
    anuncio_id INT,
    conteudo TEXT,
    lido INT DEFAULT 0,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (remetente_id) REFERENCES usuarios(id),
    FOREIGN KEY (receptor_id) REFERENCES usuarios(id)
);
```

---

## 🧪 Testar Localmente

### Com JSON Server (Rápido)

```bash
npm install -g json-server

# Criar db.json
cat > db.json << 'EOF'
{
  "usuarios": [],
  "anuncios": [],
  "transacoes": []
}
EOF

# Rodar
json-server --watch db.json --port 3000
```

### Com Node.js/Express

```bash
npm init -y
npm install express cors body-parser

# Criar server.js e rodar
node server.js
```

---

## 🚀 Deploy

### Frontend (Recomendado)
- **Vercel** - Melhor para SPA
- **Netlify** - Alternativa
- **GitHub Pages** - Sem backend

### Backend
- **Heroku** - PaaS fácil
- **Railway** - Moderno
- **AWS EC2** - Mais controle
- **DigitalOcean** - Melhor custo

---

## 📚 Recursos

- [JWT.io](https://jwt.io) - Validar tokens
- [Postman](https://www.postman.com) - Testar APIs
- [Swagger/OpenAPI](https://swagger.io) - Documentar API
- [Auth0](https://auth0.com) - Autenticação como serviço

---

**Boa integração! 🚀**
