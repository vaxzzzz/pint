# MáquinasMax - E-Commerce de Máquinas Pesadas

🚗 **Plataforma de Compra, Venda e Aluguel de Máquinas Pesadas**

## 📋 Descrição

MáquinasMax é uma plataforma e-commerce responsiva e moderna para compra, venda e aluguel de máquinas pesadas como escavadeiras, tratores, guindaste, caminhões betoneira e muito mais.

## 🎯 Funcionalidades

✅ **Catálogo de Máquinas**
- Grid responsivo com filtros por categoria e tipo (venda/aluguel)
- Busca em tempo real
- Cards com imagem, preço e avaliação

✅ **Sistema de Autenticação**
- Cadastro em 2 etapas (dados pessoais + credenciais)
- Verificação de email
- Recuperação de senha
- Validação de CPF, email e senha

✅ **Painel do Usuário**
- Meus Anúncios (máquinas ativas)
- Histórico de Negócios (vendas e aluguéis)
- Carrinho de compras
- Dados de conta

✅ **Sistema de Anúncios**
- Formulário completo para anunciar máquinas
- Upload de imagem
- Descrição detalhada
- Dados do anunciante

✅ **Chat em Tempo Real**
- Conversa com vendedor
- Animações suaves
- Respostas automáticas

✅ **Checkout**
- Resumo do pedido
- Múltiplas formas de pagamento (PIX, Boleto, Cartão)
- Interface segura

## 📁 Estrutura de Pastas

```
maquinasmax-ecommerce/
├── index.html                          # Página principal
├── css/
│   └── style.css                       # CSS consolidado (1 arquivo)
├── js/
│   └── main.js                         # JavaScript principal
├── images/                             # Imagens do projeto
├── pages/
│   ├── auth/                           # Autenticação
│   │   ├── login.html
│   │   ├── cadastro.html
│   │   ├── cadastro-credenciais.html
│   │   ├── verificar-email.html
│   │   ├── email-verificado.html
│   │   ├── esqueci-senha.html
│   │   └── senha-enviada.html
│   ├── catalogo/                       # Catálogo e detalhe
│   │   ├── venda.html
│   │   └── pagina-prestador.html
│   ├── anuncios/                       # Gerenciamento de anúncios
│   │   ├── pagina-anunciar.html
│   │   ├── pagina-meus-anuncios.html
│   │   └── pagina-meus-alugueis.html
│   └── checkout/                       # Carrinho e pagamento
│       ├── locacoes.html
│       └── pagamento.html
├── .gitignore
└── README.md                           # Este arquivo
```

## 🎨 Paleta de Cores

- **Primária:** #F7931E (Laranja)
- **Primária Escura:** #e07d0e
- **Texto:** #1a1a1a (Cinza Escuro)
- **Fundo:** #f8f9fa (Cinza Claro)
- **Sucesso:** #28a745 (Verde)
- **Erro:** #dc3545 (Vermelho)

## 🛠️ Tecnologias

- **HTML5** - Semântica e acessibilidade
- **CSS3** - Flexbox, Grid, Animações
- **JavaScript (Vanilla)** - Sem dependências externas
- **Responsive Design** - Mobile-first

## 📱 Responsividade

- **Desktop:** 1200px+
- **Tablet:** 768px - 1199px
- **Mobile:** 320px - 767px

## 🚀 Como Usar

### 1. Clonar o Repositório

```bash
git clone https://github.com/usuario/maquinasmax-ecommerce.git
cd maquinasmax-ecommerce
```

### 2. Abrir no Navegador

```bash
# Abrir index.html no navegador
open index.html
# ou
# Usar um servidor local (recomendado)
python -m http.server 8000
# Acesse: http://localhost:8000
```

### 3. Estrutura de URLs

```
/                           → Página Principal
/pages/auth/login.html      → Login
/pages/auth/cadastro.html   → Cadastro
/pages/catalogo/venda.html  → Detalhe do Produto
/pages/anuncios/            → Gerenciamento de Anúncios
/pages/checkout/            → Carrinho e Pagamento
```

## 📝 Funcionalidades JavaScript

### Validações
- `validarEmail()` - Valida formato de email
- `validarCPF()` - Valida CPF com dígitos verificadores
- `validarSenha()` - Verifica tamanho mínimo
- `validarTelefone()` - Valida telefone

### Máscaras
- `mascaraCPF()` - Formata CPF (000.000.000-00)
- `mascaraTelefone()` - Formata telefone ((00) 00000-0000)
- `mascaraCEP()` - Formata CEP (00000-000)

### Filtros
- `aplicarFiltros()` - Filtra produtos por categoria, tipo e busca
- `atualizarTitulo()` - Atualiza título dinâmico

### Chat
- `ChatManager` - Classe para gerenciar chat
- `enviarMensagem()` - Envia e processa mensagens
- `gerarRespostaAutomatica()` - Gera respostas do vendedor

### 💳 Pagamento
- `PagamentoManager` - Classe para gerenciar pagamentos
- Suporte a **PIX**, **Cartão de Crédito** e **Boleto**
- Validação completa de cartão (número, validade, CVV)
- Geração de QR Code PIX (simulado)
- Integração pronta para **MercadoPago** / **Stripe**
- Detecção automática de bandeira de cartão
- Parcelamento até 12x

## 🔒 Segurança

- Validação client-side de dados
- Mascaramento de inputs sensíveis
- CSRF Protection pronto para backend
- Sem hardcoding de senhas
- SessionStorage para dados temporários

## 🚧 Próximos Passos (Backend)

Para colocar em produção, integrar com:

1. **API REST** - Node.js/Express, Python/Flask, etc.
2. **Banco de Dados** - MongoDB, PostgreSQL, MySQL
3. **Autenticação** - JWT, OAuth 2.0
4. **Email** - SendGrid, Nodemailer
5. **Pagamentos** - Stripe, MercadoPago
6. **Armazenamento** - AWS S3, Cloudinary
7. **WebSocket** - Socket.io para chat real

## 📞 Suporte

Para dúvidas ou sugestões, abra uma issue no GitHub.

## 📄 Licença

Este projeto está sob licença MIT.

---

**Desenvolvido com ❤️ para facilitar negócios de máquinas pesadas**

Versão: 1.0.0
Data: 2025
