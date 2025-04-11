# Sistema-bancario
from abc import ABC, abstractmethod
from datetime import date, datetime

class Transacao(ABC):
    @property
    @abstractmethod
    def valor(self) -> float:
        pass

    @abstractmethod
    def registrar(self, conta) -> None:
        pass

class Historico:
    def __init__(self):
        self._transacoes = []

    @property
    def transacoes(self) -> list:
        return self._transacoes

    def adicionar_transacao(self, transacao: 'Transacao') -> None:
        self._transacoes.append({
            "tipo": transacao.__class__.__name__,
            "valor": transacao.valor,
            "data": datetime.now().strftime("%d-%m-%Y %H:%M:%S")
        })

class Conta:
    def __init__(self, cliente, numero, agencia="0001"):
        self._saldo = 0.0
        self._numero = numero
        self._agencia = agencia
        self._cliente = cliente
        self._historico = Historico()

    @classmethod
    def nova_conta(cls, cliente, numero):
        return cls(cliente, numero)

    @property
    def saldo(self) -> float:
        return self._saldo

    @property
    def numero(self) -> int:
        return self._numero

    @property
    def agencia(self) -> str:
        return self._agencia

    @property
    def cliente(self):
        return self._cliente

    @property
    def historico(self):
        return self._historico

    def sacar(self, valor: float) -> bool:
        if valor <= 0:
            print("\n@@@ Operação falhou! O valor é inválido. @@@")
            return False

        if self._saldo < valor:
            print("\n@@@ Operação falhou! Saldo insuficiente. @@@")
            return False

        self._saldo -= valor
        print("\n=== Saque realizado com sucesso! ===")
        return True

    def depositar(self, valor: float) -> bool:
        if valor <= 0:
            print("\n@@@ Operação falhou! O valor é inválido. @@@")
            return False

        self._saldo += valor
        print("\n=== Depósito realizado com sucesso! ===")
        return True

class ContaCorrente(Conta):
    def __init__(self, cliente, numero, limite=500, limite_saques=3):
        super().__init__(cliente, numero)
        self._limite = limite
        self._limite_saques = limite_saques

    def sacar(self, valor: float) -> bool:
        numero_saques = len(
            [transacao for transacao in self.historico.transacoes 
             if transacao["tipo"] == "Saque"]
        )

        excedeu_limite = valor > self._limite
        excedeu_saques = numero_saques >= self._limite_saques

        if excedeu_limite:
            print("\n@@@ Operação falhou! O valor do saque excede o limite. @@@")
            return False

        if excedeu_saques:
            print("\n@@@ Operação falhou! Número máximo de saques excedido. @@@")
            return False

        return super().sacar(valor)

class Cliente:
    def __init__(self, endereco: str):
        self._endereco = endereco
        self._contas = []

    def realizar_transacao(self, conta, transacao: Transacao):
        transacao.registrar(conta)

    def adicionar_conta(self, conta):
        self._contas.append(conta)

class PessoaFisica(Cliente):
    def __init__(self, cpf: str, nome: str, data_nascimento: date, endereco: str):
        super().__init__(endereco)
        self._cpf = cpf
        self._nome = nome
        self._data_nascimento = data_nascimento

    @property
    def cpf(self) -> str:
        return self._cpf

    @property
    def nome(self) -> str:
        return self._nome

    @property
    def data_nascimento(self) -> date:
        return self._data_nascimento

class Deposito(Transacao):
    def __init__(self, valor: float):
        self._valor = valor

    @property
    def valor(self) -> float:
        return self._valor

    def registrar(self, conta):
        sucesso = conta.depositar(self.valor)
        if sucesso:
            conta.historico.adicionar_transacao(self)

class Saque(Transacao):
    def __init__(self, valor: float):
        self._valor = valor

    @property
    def valor(self) -> float:
        return self._valor

    def registrar(self, conta):
        sucesso = conta.sacar(self.valor)
        if sucesso:
            conta.historico.adicionar_transacao(self)

def menu():
    menu = """\n
    ================ MENU ================
    [d]\tDepositar
    [s]\tSacar
    [e]\tExtrato
    [nc]\tNova conta
    [lc]\tListar contas
    [nu]\tNovo usuário
    [q]\tSair
    => """
    return input(menu)

def main():
    clientes = []
    contas = []

    while True:
        opcao = menu()

        if opcao == "d":
            numero_conta = int(input("Informe o número da conta: "))
            conta = filtrar_conta(numero_conta, contas)
            
            if not conta:
                print("\n@@@ Conta não encontrada! @@@")
                continue
                
            valor = float(input("Informe o valor do depósito: "))
            transacao = Deposito(valor)
            cliente = conta.cliente
            cliente.realizar_transacao(conta, transacao)

        elif opcao == "s":
            numero_conta = int(input("Informe o número da conta: "))
            conta = filtrar_conta(numero_conta, contas)
            
            if not conta:
                print("\n@@@ Conta não encontrada! @@@")
                continue
                
            valor = float(input("Informe o valor do saque: "))
            transacao = Saque(valor)
            cliente = conta.cliente
            cliente.realizar_transacao(conta, transacao)

        elif opcao == "e":
            numero_conta = int(input("Informe o número da conta: "))
            conta = filtrar_conta(numero_conta, contas)
            
            if not conta:
                print("\n@@@ Conta não encontrada! @@@")
                continue
                
            print("\n================ EXTRATO ================")
            transacoes = conta.historico.transacoes
            
            if not transacoes:
                print("Não foram realizadas movimentações.")
            else:
                for transacao in transacoes:
                    print(f"\n{transacao['data']} - {transacao['tipo']}: R$ {transacao['valor']:.2f}")
            
            print(f"\nSaldo: R$ {conta.saldo:.2f}")
            print("==========================================")

        elif opcao == "nu":
            cpf = input("Informe o CPF (somente números): ")
            cliente = filtrar_cliente(cpf, clientes)
            
            if cliente:
                print("\n@@@ Já existe cliente com esse CPF! @@@")
                continue
                
            nome = input("Informe o nome completo: ")
            data_nascimento = input("Informe a data de nascimento (dd-mm-aaaa): ")
            endereco = input("Informe o endereço (logradouro, nro - bairro - cidade/sigla estado): ")
            
            try:
                dia, mes, ano = map(int, data_nascimento.split('-'))
                data_nascimento = date(ano, mes, dia)
            except:
                print("\n@@@ Data de nascimento inválida! @@@")
                continue
                
            cliente = PessoaFisica(
                cpf=cpf, 
                nome=nome, 
                data_nascimento=data_nascimento, 
                endereco=endereco
            )
            
            clientes.append(cliente)
            print("\n=== Cliente criado com sucesso! ===")

        elif opcao == "nc":
            cpf = input("Informe o CPF do cliente: ")
            cliente = filtrar_cliente(cpf, clientes)
            
            if not cliente:
                print("\n@@@ Cliente não encontrado! @@@")
                continue
                
            numero_conta = len(contas) + 1
            conta = ContaCorrente.nova_conta(cliente=cliente, numero=numero_conta)
            contas.append(conta)
            cliente.adicionar_conta(conta)
            print("\n=== Conta criada com sucesso! ===")

        elif opcao == "lc":
            listar_contas(contas)

        elif opcao == "q":
            break

        else:
            print("\n@@@ Operação inválida, por favor selecione novamente a operação desejada. @@@")

def filtrar_cliente(cpf: str, clientes: list):
    for cliente in clientes:
        if hasattr(cliente, 'cpf') and cliente.cpf == cpf:
            return cliente
    return None

def filtrar_conta(numero_conta: int, contas: list):
    for conta in contas:
        if conta.numero == numero_conta:
            return conta
    return None

def listar_contas(contas: list):
    for conta in contas:
        print("=" * 100)
        print(f"Agência:\t{conta.agencia}")
        print(f"C/C:\t\t{conta.numero}")
        print(f"Titular:\t{conta.cliente.nome}")

if __name__ == "__main__":
    main()
