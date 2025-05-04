# ANALISE-DE-DADOS-CURSO-ASIMOV
Python analise de dados

````py
caminho_dados = 'C:\\ARQUIVOS\\asimov\\Conteeudo\\2023_Viagem.csv'
caminho_saida_tabela = 'C:\\ARQUIVOS\\asimov\\Conteeudo\\tabela_2023.xlsx'
caminho_saida_grafico = 'C:\\ARQUIVOS\\asimov\\Conteeudo\\grafico_2023.png'

import pandas as pd

pd.set_option('display.max_columns', None)
pd.set_option('display.float_format', '{:.2f}'.format)

#lendo os dados
df_viagens = pd.read_csv(caminho_dados, encoding='Windows-1252', sep=';', decimal= ',')

#criando nova coluna despesas

df_viagens['Despesas'] = df_viagens['Valor diárias'] + df_viagens['Valor passagens'] + df_viagens['Valor outros gastos']

#Ajustando valores nullos no campo Cargo
df_viagens['Cargo'] = df_viagens['Cargo'].fillna("Não identificado")

#convertendo colunas de datas
df_viagens["Período - Data de início"] = pd.to_datetime(df_viagens["Período - Data de início"], format="%d/%m/%Y")
df_viagens["Período - Data de fim"] = pd.to_datetime(df_viagens["Período - Data de fim"], format="%d/%m/%Y")

#Criando novas colunas de datas
df_viagens['Mês da viagem'] = df_viagens['Período - Data de início'].dt.month_name()
df_viagens['Dias de viagem'] = (df_viagens['Período - Data de fim'] - df_viagens['Período - Data de início']).dt.days

#criando tabela consolidada
df_viagens_consolidado = (df_viagens
    .groupby('Cargo')
    .agg(
        despesa_media=('Despesas', 'mean'),
        duracao_media=('Dias de viagem', 'mean'),
        despesas_totais=('Despesas', 'sum'),
        destino_mais_frequente=('Destinos', pd.Series.mode),
        n_viagens=('Nome', 'count')
    )
    .reset_index()
    .sort_values(by='despesas_totais', ascending=False))

#Filtrando tabela consolidada por cargos relevantes(> 1% das viagens)

df_cargos = df_viagens['Cargo'].value_counts(normalize=True).reset_index()
cargos_relevantes = df_cargos.loc[df_cargos['proportion'] > 0.01, 'Cargo']
filtro = df_viagens_consolidado['Cargo'].isin(cargos_relevantes)

#chegando na tabela final = consolidada e filtrada
df_final = df_viagens_consolidado[filtro].sort_values(by='n_viagens', ascending=False)

#Salvando a tabela final

df_final.to_excel(caminho_saida_tabela, index=False)


import matplotlib.pyplot as plt
#criando a figura
fig, ax = plt.subplots(figsize = (16, 6))

#plotando o gráfico
ax.barh(df_final['Cargo'], df_final['despesa_media'], color="#49deac")
ax.invert_yaxis()

#ajustando o gráfico
ax.set_facecolor('#3b3b47')
fig.suptitle('Despesa média em viagens por cargo público(2023)')
plt.figtext(0.65, 0.89, 'Fonte: Portal da Transparencia', fontsize=8)
plt.grid(color='gray', linestyle= '--', linewidth=0.5)
plt.yticks(fontsize=8)
plt.xlabel('Despesa média')

#salvando o grafico
plt.savefig(caminho_saida_grafico, bbox_inches='tight')

````
