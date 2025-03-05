import wikipediaapi
import re
import pyttsx3  # Biblioteca para fazer o Javis falar
import datetime
import webbrowser
import random
import requests
import urllib.parse

# Inicializa o sintetizador de voz
javis_voz = pyttsx3.init()
javis_voz.setProperty('rate', 180)  # Velocidade da fala
javis_voz.setProperty('volume', 1.0)  # Volume máximo

# Configurações da API do Google Search
GOOGLE_API_KEY = "sua chave"
GOOGLE_CX = "sua chave"
NEWS_API_KEY = "sua chave"

def falar(texto):
    """Faz o Javis falar."""
    print(f"🎙 Javis: {texto}")
    javis_voz.say(texto)
    javis_voz.runAndWait()

def limpar_texto(texto):
    """Remove informações desnecessárias, como referências e formatações estranhas."""
    texto = re.sub(r'\[\d+\]', '', texto)  
    return texto.strip()

def buscar_wikipedia(pergunta):
    user_agent = "JavisBot/1.0 (seuemail@example.com)"  
    wiki = wikipediaapi.Wikipedia(user_agent=user_agent, language='pt')  

    pagina = wiki.page(pergunta)

    if not pagina.exists():
        return buscar_google(pergunta)

    titulo = pagina.title
    texto_bruto = pagina.summary  
    texto_limpo = limpar_texto(texto_bruto)
    
    resposta = " ".join(texto_limpo.split(".")[:5]) + "."  # Pegando as primeiras 5 frases
    
    falar(resposta)
    return f"📌 {titulo}\n\n{resposta}\n\n🔗 Leia mais: {pagina.fullurl}"

def buscar_google(pergunta):
    """Busca informações no Google e retorna uma única resposta detalhada."""
    query = urllib.parse.quote(pergunta)
    url = f"https://www.googleapis.com/customsearch/v1?q={query}&key={GOOGLE_API_KEY}&cx={GOOGLE_CX}"
    resposta = requests.get(url).json()
    resultados = resposta.get("items", [])  
    
    if not resultados:
        falar("Não encontrei nada relevante.")
        return "Nenhum resultado encontrado."
    
    resposta_final = resultados[0]["snippet"]  # Pegando apenas a primeira resposta encontrada
    
    falar(resposta_final)
    return f"📌 {resposta_final}\n\n🔗 Leia mais: {resultados[0]['link']}"

def dizer_hora():
    agora = datetime.datetime.now()
    resposta = f"Agora são {agora.hour} horas e {agora.minute} minutos."
    falar(resposta)
    return resposta

def abrir_site(url):
    webbrowser.open(url)
    falar(f"Abrindo {url}")

def contar_piada():
    piadas = [
        "O que o pato disse para a pata? Vem Quá!",
        "Por que o livro de matemática ficou triste? Porque tinha muitos problemas!",
        "O que o zero disse para o oito? Que cinto bonito!"
    ]
    piada = random.choice(piadas)
    falar(piada)
    return piada

def ler_noticias():
    url = f"https://newsapi.org/v2/top-headlines?country=br&apiKey={NEWS_API_KEY}"
    resposta = requests.get(url).json()
    noticias = resposta.get("articles", [])  
    
    if not noticias:
        falar("Desculpe, não encontrei notícias no momento.")
        return "Nenhuma notícia encontrada."
    
    noticia = noticias[0]  # Pegando apenas a primeira notícia
    titulo = noticia["title"]
    descricao = noticia.get("description", "Sem descrição disponível.")
    resposta_final = f"{titulo} - {descricao}"
    
    falar(resposta_final)
    return f"📌 {resposta_final}\n\n🔗 Leia mais: {noticia['url']}"

# 🚀 Javis em ação
falar("Em que posso ajudar hoje, Luis")
while True:
    pergunta = input("📝 Você: ").lower()
    
    if pergunta in ["sair", "fechar", "tchau"]:
        falar("Até logo! Se precisar de mim, estarei por aqui.")
        break
    elif "horas" in pergunta or "que horas são" in pergunta:
        print(dizer_hora())
    elif "abra o google" in pergunta:
        abrir_site("https://www.google.com")
    elif "piada" in pergunta or "conte uma piada" in pergunta:
        print(contar_piada())
    elif "notícias" in pergunta or "noticias do dia" in pergunta:
        print(ler_noticias())
    else:
        print(buscar_wikipedia(pergunta))
