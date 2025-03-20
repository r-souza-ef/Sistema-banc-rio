# Sistema-bancario
# Variáveis globais
saldo = 0.0
extrato = []
numero_saques = 0
LIMITE_SAQUES = 3
LIMITE_VALOR_SAQUE = 500.00

#Função para depósito
def depositar (valor):
    global saldo, extrato
    if valor > 0:
        saldo += valor
        extrato.append(f"Depósito: R${valor:.2f}")
        print(f"Depósito de R$ {valor:.2f} realizado com sucesso.")
    else:
        print("Valor inválido para depósito. O valor deve ser positivo.")

#Função para saque
def sacar(valor):
    global saldo, extrato, numero_saques
    if valor > 0:
        if numero_saques < LIMITE_SAQUES:
            if valor <= LIMITE_VALOR_SAQUE:
                if saldo >= valor:
                    saldo -= valor
                    extrato.append(f"Saque: R$ {valor:.2f}")
                    numero_saques += 1
                    print (f"Saque de R$ {valor:.2f} realizado com sucesso.")
                else:
                    print ("Saldo insuficiente para realizar o saque.")
            else:
                print (f"O valor máximo por saque é de R$ {LIMITE_VALOR_SAQUE: .2f}.")

#Função para exibir extrato
def exibir_extrato ():
    global saldo, extrato
    print ("\n========== EXTRATO ==========")
    if not extrato:
        print("Não foram realizados movimentações.")
    else:
        for movimento in extrato:
            print (movimento)
        print(f"\nSaldo atual: R$ {saldo:.2f}")
        print("=============================")

#Menu principal
def main():
    while True:
        print("\n========== MENU ==========")
        print ("1. Depositar")
        print ("2. Sacar")
        print ("3. Extrato")
        print ("4. Sair")
        opcao = input("Escolha uma opção: ")
        
        if opcao == "1":
            valor = float(input("Digite o valor para depósito: "))
            depositar(valor)
        elif opcao == "2":
            valor = float(input("Digite o valor para saque: "))
            sacar(valor)
        elif opcao == "3":
            exibir_extrato()
        elif opcao == "4":
            print("Saindo do sistema...")
            break
        else:
            print("Opção invalida. Tente novamente.")

#Executar o programa
if __name__ == "__main__":
    main ()
