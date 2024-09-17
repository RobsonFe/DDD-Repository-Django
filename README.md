# Projeto: Integração de Domain-Driven Design (DDD) e Repository Pattern no Django

Este projeto demonstra como utilizar Domain-Driven Design (DDD) em conjunto com o padrão de repositório no Django, fornecendo uma estrutura organizada para sistemas complexos. Ele destaca a separação entre o domínio e a infraestrutura, facilitando a manutenção e a evolução do sistema ao longo do tempo.

### O que é Domain-Driven Design (DDD)?

DDD é uma abordagem para design de software que coloca o foco no domínio de negócios e em como ele pode ser modelado no código. Seus principais conceitos incluem:

- **Entidades**: Objetos com uma identidade única.
- **Objetos de Valor (Value Objects)**: Objetos definidos apenas por seus atributos.
- **Agregados**: Conjuntos de entidades e objetos de valor tratados como uma unidade.
- **Repositórios**: Interfaces para acessar agregados.
- **Serviços**: Regras de negócio que não pertencem diretamente a entidades ou objetos de valor.

### O que é o Padrão de Repositório?

O padrão de repositório atua como uma camada de abstração entre a lógica de negócio e a camada de persistência. Ele ajuda a isolar a lógica de negócios das operações de banco de dados, promovendo uma arquitetura mais limpa e flexível.

### Estrutura do Projeto

Abaixo está uma sugestão de como você pode organizar o projeto Django para aplicar DDD e o padrão de repositório:

```
meu_projeto/
│
├── manage.py
├── meu_projeto/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── dominio/
│   ├── __init__.py
│   ├── entidades/
│   │   ├── __init__.py
│   │   └── produto.py
│   ├── repositorios/
│   │   ├── __init__.py
│   │   └── produto_repository.py
│   ├── servicos/
│   │   ├── __init__.py
│   │   └── produto_service.py
│   └── valores/
│       ├── __init__.py
│       └── preco.py
│
├── app_principal/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py
│   └── migrations/
│       └── __init__.py
└── requirements.txt
```

### Exemplo de Implementação

#### Entidade: Produto (dominio/entidades/produto.py)

```python
class Produto:
    def __init__(self, id, nome, preco, descricao):
        self.id = id
        self.nome = nome
        self.preco = preco
        self.descricao = descricao

    def atualizar_preco(self, novo_preco):
        self.preco = novo_preco
```

#### Repositório: ProdutoRepository (dominio/repositorios/produto_repository.py)

```python
from app_principal.models import Produto as ProdutoModel

class ProdutoRepository:
    def get_all(self):
        return [self._to_entity(produto) for produto in ProdutoModel.objects.all()]

    def get_by_id(self, id):
        produto = ProdutoModel.objects.get(id=id)
        return self._to_entity(produto)

    def save(self, produto):
        produto_model = self._to_model(produto)
        produto_model.save()

    def delete(self, id):
        ProdutoModel.objects.get(id=id).delete()

    def _to_entity(self, produto_model):
        return Produto(produto_model.id, produto_model.nome, produto_model.preco, produto_model.descricao)

    def _to_model(self, produto):
        return ProdutoModel(id=produto.id, nome=produto.nome, preco=produto.preco, descricao=produto.descricao)
```

#### Serviço: ProdutoService (dominio/servicos/produto_service.py)

```python
from dominio.repositorios.produto_repository import ProdutoRepository

class ProdutoService:
    def __init__(self):
        self.repository = ProdutoRepository()

    def listar_todos(self):
        return self.repository.get_all()

    def criar_produto(self, nome, preco, descricao):
        produto = Produto(None, nome, preco, descricao)
        self.repository.save(produto)

    def atualizar_produto(self, id, nome, preco, descricao):
        produto = self.repository.get_by_id(id)
        produto.nome = nome
        produto.atualizar_preco(preco)
        produto.descricao = descricao
        self.repository.save(produto)

    def deletar_produto(self, id):
        self.repository.delete(id)
```

### Considerações

- **Separação de Domínio e Infraestrutura**: O modelo de domínio, composto por entidades, repositórios e serviços, é separado da infraestrutura de persistência (Django ORM, views, etc.).
- **Flexibilidade**: A lógica de negócios é desacoplada da tecnologia de persistência, permitindo alterações futuras sem impacto significativo no código de negócio.
- **Complexidade**: Essa abordagem pode ser mais complexa do que o necessário para projetos simples. Deve ser usada quando a modelagem do domínio é central para o sistema.
