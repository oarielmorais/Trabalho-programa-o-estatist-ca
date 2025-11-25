[ruina_jogador_Version2 (1).py](https://github.com/user-attachments/files/23738039/ruina_jogador_Version2.1.py)
# Ruina do Jogador - Simulação Completa
# Trabalho de programação | Código unificado

# ------------------------------
# 1. Importação de Bibliotecas
# ------------------------------
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# ------------------------------
# 2. CONFIGURAÇÃO INICIAL E MODELO
# ------------------------------
np.random.seed(42)  # Fixando semente para reproduzibilidade

def simular_jogo(capital_inicial, meta, prob_ganhar):
    """
    Simula uma trajetória do jogo até ruína ou sucesso
    """
    capital = capital_inicial
    tempo = 0
    while capital > 0 and capital < meta:
        capital += 1 if np.random.random() < prob_ganhar else -1
        tempo += 1
    return "ruína" if capital == 0 else "sucesso", tempo

# ------------------------------
# 3. PLANO EXPERIMENTAL
# ------------------------------
META = 50
REPLICACOES = 10000
CAPITAIS = [5, 10, 25, 45]
PROBABILIDADES = [0.5, 0.49]

print("=== SIMULAÇÃO DA RUÍNA DO JOGADOR ===")
print(f"Meta: {META}, Replicações: {REPLICACOES}")
print(f"Capitais: {CAPITAIS}, Probabilidades: {PROBABILIDADES}")

# ------------------------------
# 4. EXECUÇÃO DOS EXPERIMENTOS
# ------------------------------
resultados = []

for capital in CAPITAIS:
    for p in PROBABILIDADES:
        ruinass = 0
        duracoes = []

        for _ in range(REPLICACOES):
            resultado, duracao = simular_jogo(capital, META, p)
            if resultado == "ruína":
                ruinass += 1
            duracoes.append(duracao)

        prob_ruina = ruinass / REPLICACOES
        media_duracao = np.mean(duracoes)
        dp_duracao = np.std(duracoes)

        resultados.append({
            'capital': capital,
            'prob_ganhar': p,
            'tipo': 'Justo' if p == 0.5 else 'Viés 1%',
            'prob_ruina': prob_ruina,
            'media_duracao': media_duracao,
            'dp_duracao': dp_duracao
        })

# ------------------------------
# 5. TABELA DE RESULTADOS
# ------------------------------
df = pd.DataFrame(resultados)
print("\n" + "="*60)
print("RESULTADOS COMPLETOS")
print("="*60)

tabela = df[['capital', 'tipo', 'prob_ruina', 'media_duracao']].copy()
tabela['prob_ruina'] = tabela['prob_ruina'].round(4)
tabela['media_duracao'] = tabela['media_duracao'].round(1)
print(tabela.to_string(index=False))

# ------------------------------
# 6. GRÁFICO 1 - Probabilidade de Ruína
# ------------------------------
plt.figure(figsize=(8, 6))
ax = plt.gca()

for tipo in ['Justo', 'Viés 1%']:
    dados = df[df['tipo'] == tipo]
    ax.plot(dados['capital'], dados['prob_ruina'], 'o-', label=tipo, markersize=8)

ax.set_xlabel('Capital Inicial')
ax.set_ylabel('Probabilidade de Ruína')
ax.set_title('Probabilidade de Ruína vs Capital Inicial')
ax.legend()
ax.grid(True, alpha=0.3)
ax.set_xticks(CAPITAIS)

plt.show()

# ------------------------------
# 7. GRÁFICO 2 - Duração Média do Jogo
# ------------------------------
plt.figure(figsize=(10, 8))
for tipo in ['Justo', 'Viés 1%']:
    dados = df[df['tipo'] == tipo]
    plt.plot(dados['capital'], dados['media_duracao'], 's-', label=tipo, markersize=8)

plt.xlabel('Capital Inicial')
plt.ylabel('Duração Média (rodadas)')
plt.title('Duração Média do Jogo')
plt.legend()
plt.grid(True, alpha=0.3)
plt.xticks(CAPITAIS)

plt.tight_layout()
plt.show()

# ------------------------------
# 8. GRÁFICO 3 - Impacto do Viés de 1%
# ------------------------------
plt.figure(figsize=(8, 6))
ax = plt.gca()

impactos = []
for capital in CAPITAIS:
    justo = df[(df['capital'] == capital) & (df['tipo'] == 'Justo')]['prob_ruina'].values
    vies = df[(df['capital'] == capital) & (df['tipo'] == 'Viés 1%')]['prob_ruina'].values

    if len(justo) > 0 and len(vies) > 0 and justo[0] != 0:
        justo_val = justo[0]
        vies_val = vies[0]
        impacto = ((vies_val - justo_val) / justo_val) * 100
    else:
        impacto = 0 # Evita divisão por zero

    impactos.append(impacto)

bars = ax.bar(CAPITAIS, impactos, color='orange', alpha=0.7, width=2)
ax.set_xlabel('Capital Inicial')
ax.set_ylabel('Aumento Percentual (%)')
ax.set_title('Impacto do Viés de 1% na Probabilidade de Ruína')
ax.set_xticks(CAPITAIS)

for bar, impacto in zip(bars, impactos):
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 1,
             f'{impacto:.1f}%', ha='center', va='bottom')

plt.show()

# ------------------------------
# 9. GRÁFICO 4 - Trajetórias Exemplares
# ------------------------------
plt.figure(figsize=(8, 6))
ax = plt.gca()

np.random.seed(123)
META = 50

for i in range(4):
    capital = 25
    historico = [capital]

    while capital > 0 and capital < META and len(historico) < 500:
        capital += 1 if np.random.random() < 0.5 else -1
        historico.append(capital)

    ax.plot(historico, linewidth=1, alpha=0.7, label=f'Traj {i+1}')

ax.axhline(y=0, color='red', linestyle='--', label='Ruína')
ax.axhline(y=META, color='green', linestyle='--', label='Sucesso')
ax.set_xlabel('Rodadas')
ax.set_ylabel('Capital')
ax.set_title(f'Trajetórias Exemplares (Capital = 25, Jogo Justo)')
ax.legend()
ax.grid(True, alpha=0.3)

plt.show()

# ------------------------------
# 10. ANÁLISE E CONCLUSÕES FINAIS
# ------------------------------
print("\n1. IMPACTO DO CAPITAL INICIAL:")
for capital in CAPITAIS:
    prob = df[(df['capital'] == capital) & (df['tipo'] == 'Justo')]['prob_ruina'].values[0]
    print(f"   Capital {capital}: {prob:.1%} de chance de ruína")

print("\n2. IMPACTO DO VIÉS DE 1%:")
for capital in CAPITAIS:
    justo = df[(df['capital'] == capital) & (df['tipo'] == 'Justo')]['prob_ruina'].values[0]
    vies = df[(df['capital'] == capital) & (df['tipo'] == 'Viés 1%')]['prob_ruina'].values[0]
    aumento = ((vies - justo) / justo) * 100
    print(f"   Capital {capital}: {aumento:+.1f}% de aumento")

max_duracao = df.loc[df['media_duracao'].idxmax()]
print(f"\n3. JOGO MAIS LONGO:")
print(f"   Capital {max_duracao['capital']} ({max_duracao['tipo']}): {max_duracao['media_duracao']:.0f} rodadas")

print("\n4. CONCLUSÕES FINAIS:")
print("   • Capital inicial determina fortemente o risco de ruína")
print("   • Viés de 1% aumenta dramaticamente a probabilidade de ruína")
print("   • Capitais intermediários geram jogos mais longos")
print("   • Pequenas desvantagens acumulam-se no longo prazo")
