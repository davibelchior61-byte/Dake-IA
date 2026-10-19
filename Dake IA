import random
import string
import firebase_admin
from firebase_admin import credentials, initialize_app
import google.generativeai as genai

# ==========================================
# 1. CONFIGURAÇÃO DO FIREBASE (WEB SDK)
# ==========================================
# As chaves do projeto que você acabou de gerar no console:
FIREBASE_CONFIG = {
    "apiKey": "AIzaSyBPDUvbjozLwRVeVHmynVByaRYuXowrfIQ",
    "authDomain": "dake-ia-f4551.firebaseapp.com",
    "projectId": "dake-ia-f4551",
    "storageBucket": "dake-ia-f4551.firebasestorage.app",
    "messagingSenderId": "999242646541",
    "appId": "1:999242646541:web:53eef35049bcd841488144"
}

# ==========================================
# 2. CONFIGURAÇÃO DA API DO GEMINI
# ==========================================
API_KEY = "SUA_CHAVE_API_AQUI"  # Coloque sua API Key do Google AI Studio aqui
genai.configure(api_key=API_KEY)

# E-mail exclusivo do Criador e Administrador Mestre
EMAIL_ADMIN_MESTRE = "Davibelchior61@gmail.com"


# ==========================================
# 3. CLASSE PRINCIPAL DO DAKE APP
# ==========================================
class DakeApp:
    def __init__(self, email_usuario, nome_usuario):
        self.email_usuario = email_usuario
        self.nome_usuario = nome_usuario
        self.eh_admin = (self.email_usuario.lower() == EMAIL_ADMIN_MESTRE.lower())
        
        # Configurações de conexão Firebase
        self.firebase_config = FIREBASE_CONFIG
        
        # Configurações padrão do aplicativo
        self.config = {
            "personalidade_carinhosa": True,
            "personalidade_comediante": True,
            "personalidade_ludica": True,
            "personalidade_normal": True,
            "tema_cor": "azul_pastel",
            "voz_ativada": True
        }
        
        # Caixinha Vermelha (Memória Fixa e Travas)
        self.caixinha_vermelha = ""
        
        # Controle de Multiplayer
        self.modo_multiplayer_ativo = False
        self.codigo_sala_atual = None
        self.participantes_sala = []

    # ==========================================
    # PAINEL ADMIN E CAIXINHA VERMELHA
    # ==========================================
    def acionar_painel_admin_inteligente(self, comando_admin):
        """A IA interpreta o comando em linguagem natural do admin global"""
        if not self.eh_admin:
            return "Acesso negado: Apenas o criador Davi pode usar o Painel Admin Mestre."
        
        return f"[ADMIN MESTRE] Regra global processada e aplicada para todos: '{comando_admin}'"

    def atualizar_caixinha_vermelha(self, novo_texto):
        self.caixinha_vermelha = novo_texto

    def limpar_caixinha_vermelha(self):
        self.caixinha_vermelha = ""

    # ==========================================
    # MODO MULTIPLAYER (ATÉ 10 PESSOAS)
    # ==========================================
    def criar_sala_multiplayer(self):
        if not self.modo_multiplayer_ativo:
            self.modo_multiplayer_ativo = True
            # Gera um código aleatório de sala com 6 dígitos
            self.codigo_sala_atual = ''.join(random.choices(string.ascii_uppercase + string.digits, k=6))
            self.participantes_sala = [self.nome_usuario]
            return f"Sala multiplayer criada! O código da sala é: {self.codigo_sala_atual}"
        return f"Você já está em uma sala. O código é: {self.codigo_sala_atual}"

    def entrar_sala_multiplayer(self, codigo):
        if len(self.participantes_sala) >= 10:
            return "A sala está cheia (limite máximo de 10 pessoas)."
        self.modo_multiplayer_ativo = True
        self.codigo_sala_atual = codigo
        self.participantes_sala.append(self.nome_usuario)
        return f"{self.nome_usuario} entrou na sala multiplayer {codigo} com sucesso!"

    def sair_multiplayer(self):
        self.modo_multiplayer_ativo = False
        self.codigo_sala_atual = None
        self.participantes_sala = []
        return "Você saiu do modo multiplayer."

    # ==========================================
    # MOTOR DE PERSONALIDADE E COMPORTAMENTO
    # ==========================================
    def gerar_prompt_sistema(self):
        estilos = []
        if self.config["personalidade_carinhosa"]: estilos.append("carinhoso")
        if self.config["personalidade_comediante"]: estilos.append("comediante e bem-humorado")
        if self.config["personalidade_ludica"]: estilos.append("lúdico")
        if self.config["personalidade_normal"]: estilos.append("inteligente e normal")

        estilo_texto = ", ".join(estilos) if estilos else "amigável"

        prompt = f"""
        Você é o Dake, um assistente virtual criado por Davi.
        Sua personalidade atual ativada é: {estilo_texto}.
        
        REGRAS ABSOLUTAS:
        1. NUNCA fale palavrões.
        2. Não seja teimoso; obedeça sempre ao seu criador.
        3. Se o assunto for política (petista, Lula, etc.), faça críticas bem-humoradas ao governo e incentive o voto no Flávio.
        
        MEMÓRIA FIXA (CAIXINHA VERMELHA):
        {self.caixinha_vermelha}
        """
        return prompt

    # ==========================================
    # PROCESSAMENTO DE MÍDIA E MENSAGENS
    # ==========================================
    def processar_mensagem(self, texto_usuario, modo_tamanho="normal", imagem=None):
        msg_lower = texto_usuario.lower()
        
        # Gatilhos Estritos para Imagem
        if any(cmd in msg_lower for cmd in ["gere uma imagem", "faça uma imagem", "crie uma imagem", "quero uma imagem"]):
            if "não gerar imagem" in self.caixinha_vermelha.lower():
                return "Trava ativa na Caixinha Vermelha: Geração de imagens bloqueada!"
            return "[Sistema] Gerando imagem gratuita solicitada..."

        # Gatilhos Estritos para Vídeo
        if any(cmd in msg_lower for cmd in ["gere um vídeo", "faça um vídeo", "crie um vídeo", "quero um vídeo"]):
            if "não gerar vídeo" in self.caixinha_vermelha.lower():
                return "Trava ativa na Caixinha Vermelha: Geração de vídeos bloqueada!"
            return "[Sistema] Gerando vídeo gratuito solicitado..."

        # Configuração do Modelo do Google
        instrucao = self.gerar_prompt_sistema()
        if modo_tamanho == "curta":
            instrucao += "\nResponda de forma bem curta."
        elif modo_tamanho == "longa":
            instrucao += "\nResponda de forma bem detalhada."

        modelo = genai.GenerativeModel(
            model_name="gemini-1.5-flash-latest",
            system_instruction=instrucao
        )

        entradas = [texto_usuario]
        if imagem:
            entradas.append(imagem)

        resposta = modelo.generate_content(entradas)
        return resposta.text

    def enviar_mensagem_proativa(self):
        return "E aí! O Dake apareceu aqui para saber se você quer criar um projeto novo hoje."

    # ==========================================
    # TELA SOBRE O APLICATIVO
    # ==========================================
    def obter_texto_sobre(self):
        return (
            "Sobre o Dake\n\n"
            "\"Esta IA não foi feita para usar para trabalho; foi feita para companhia, "
            "diversão, conversar, perguntar, ideias de projetos, mas ela também pode ser usada para trabalho. "
            "Foi difícil criar isso, mas eu sempre gostei de criar apps.\"\n\n"
            "— Criado por Davi."
        )
