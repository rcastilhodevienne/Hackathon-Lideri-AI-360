import requests
from datetime import datetime

# Variáveis globais (URLs das APIs financeiras e do Dify)
DIFY_API_URL = "https://dify.ip2cloud.com.br/api"
FINANCE_API_URL = "https://api.exemplo.com/finance"
TECH_API_URL = "https://api.exemplo.com/tech"

# Autenticação Dify (chave de API, etc.)
DIFY_AUTH = {
    'user': 'hackathon@ip2cloud.com.br',
    'password': 'hWN@p^7r7%q^DX!z$OPK'
}

# Função para gerar uma resposta humanizada usando a API do Dify
def generate_humanized_response(prompt):
    headers = {
        'Authorization': f"Bearer {DIFY_AUTH['password']}",
        'Content-Type': 'application/json'
    }
    data = {
        "messages": [{"role": "user", "content": prompt}]
    }
    
    response = requests.post(f"{DIFY_API_URL}/chat-messages", json=data, headers=headers)
    
    if response.status_code == 200:
        return response.json()["choices"][0]["message"]["content"]
    else:
        return "Desculpe, não consegui gerar uma resposta no momento."

# Função para consultar faturas financeiras
def get_financial_info(client_id):
    response = requests.get(f"{FINANCE_API_URL}/ListaFaturasCliente/{client_id}")
    
    if response.status_code == 200:
        return response.json()
    else:
        return {"error": "Não foi possível consultar as faturas no momento."}

# Função para imprimir a fatura solicitada
def print_invoice(invoice_id):
    response = requests.get(f"{FINANCE_API_URL}/ImprimeFatura/{invoice_id}")
    
    if response.status_code == 200:
        return response.content
    else:
        return "Erro ao imprimir a fatura."

# Função para consultar status técnico das ONUs/MODEMs
def get_technical_status(onu_id):
    response = requests.get(f"{TECH_API_URL}/status/{onu_id}")
    
    if response.status_code == 200:
        return response.json()
    else:
        return {"error": "Erro ao consultar o status do equipamento."}

# Função principal de atendimento (exemplo: interação com cliente)
def atendimento_cliente(client_id, query_type, onu_id=None, invoice_id=None):
    if query_type == "financeiro":
        # Exemplo de consulta financeira
        faturas = get_financial_info(client_id)
        if "error" in faturas:
            return generate_humanized_response("Desculpe, houve um problema ao acessar suas faturas.")
        else:
            response_text = "Aqui estão suas faturas:\n"
            for fatura in faturas:
                response_text += f"Fatura {fatura['id']}: Vencimento em {fatura['vencimento']} - Valor: {fatura['valor']}\n"
            return generate_humanized_response(response_text)
    
    elif query_type == "impressao_fatura" and invoice_id:
        # Impressão de fatura solicitada
        fatura = print_invoice(invoice_id)
        return generate_humanized_response(f"Sua fatura foi impressa com sucesso:\n{fatura}")
    
    elif query_type == "tecnico" and onu_id:
        # Exemplo de consulta técnica de equipamentos
        status = get_technical_status(onu_id)
        if "error" in status:
            return generate_humanized_response("Desculpe, houve um problema ao acessar os dados do equipamento.")
        else:
            response_text = (
                f"Status do equipamento ONU/Modem:\n"
                f"ID: {status['id']}\n"
                f"Estado: {status['state']}\n"
                f"Última vez online: {status['UpTime']}\n"
                f"Potência RX/TX: {status['RxPower']} / {status['TxPower']}\n"
                f"Distância: {status['Distance']} metros\n"
            )
            return generate_humanized_response(response_text)

    else:
        return generate_humanized_response("Desculpe, não consegui entender sua solicitação.")


# Exemplo de execução da função de atendimento
if __name__ == "__main__":
    cliente_id = "12345"
    tipo_solicitacao = "financeiro"  # ou "tecnico"
    resultado = atendimento_cliente(cliente_id, tipo_solicitacao)
    print(resultado)
