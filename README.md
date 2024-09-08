import os
import pandas as pd
import tkinter as tk
from tkinter import filedialog, messagebox

def primeiro_codigo():
    # Chame o primeiro código no final da função do segundo código
 import os  # noqa: F401
import pandas as pd  # noqa: E402, F811
import tkinter as tk  # noqa: E402, F811
from tkinter import filedialog, messagebox  # noqa: E402, F811

caminho_arquivos = r'C:\Users\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""  # Variável global para armazenar o nome do arquivo
base0150_path = ""  # Variável global para armazenar o caminho da base 0150 

def iniciar():
    global arquivo
    global base0150_path

    def abrir_janela(texto):
        janela_popup = tk.Tk()
        texto_editavel = tk.Text(janela_popup)
        texto_editavel.insert(tk.END, texto)
        texto_editavel.pack()
        botao_ok = tk.Button(janela_popup, text="Confirmar", command=lambda: [janela_popup.destroy(), janela.quit()])  # Encerra a janela popup e a janela principal
        botao_ok.pack()
        janela_popup.mainloop()

    codificacoes = ['utf-8', 'latin1', 'iso-8859-1']  # Adicione aqui as codificações que você deseja tentar
    for cod in codificacoes:
        try:
            with open(arquivo, 'r', encoding='iso-8859-1') as f:
                linhas = f.readlines()
            print(f"Arquivo lido com sucesso usando a codificação {cod}!")
            break
        except UnicodeDecodeError:
            print(f"Falha ao ler o arquivo usando a codificação {cod}. Tentando a próxima...")
        except FileNotFoundError:
         abrir_janela("Arquivo não encontrado.")
        return

    registros_0150 = [linha.strip().split('|') for linha in linhas if linha.startswith('0150')]
    registros_5015 = [linha.strip().split('|') for linha in linhas if linha.startswith('5015')]

    tabela1 = pd.DataFrame(registros_0150, columns=['REG', 'COD_PART', 'NOME', 'COD_PAIS', 'CNPJ', 'IE', 'UF', 'CEP', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'FONE'])
    cod_part1 = tabela1['COD_PART'].str.strip()

    tabela2 = pd.DataFrame(registros_5015, columns=['REG', 'NUM_LANC', 'DT_MOV', 'HIST', 'TIP_DOC', 'SER', 'NUM_DOC', 'CFOP', 'NUM_DI', 'COD_PART', 'COD_LANC', 'IND', 'COD_ITEM_OUTRA_T', 'QUAN', 'CUST_MERC', 'VL_ICMS'])
    cod_part2 = tabela2['COD_PART'].str.strip()
    cod_part2 = cod_part2.drop_duplicates()

    diferenca = cod_part2[~cod_part2.isin(cod_part1)]
    print(f"Diferença: {diferenca}")

    # Alteração: agora a base0150 é lida do arquivo selecionado pelo usuário
    base0150 = pd.read_excel(base0150_path, dtype={3: str, 'CNPJ': str, 'IE': str, 'UF': str, 'CEP': str, 'FONE': str})

    cnpjs_processados = set()
    novos_registros = []  # Lista para armazenar os novos registros 0150

    for cnpj in diferenca.tolist():
        if cnpj in base0150['CNPJ'].values:
            if cnpj in cnpjs_processados:
                continue
            cnpjs_processados.add(cnpj)
            fornecedor = base0150.loc[base0150['CNPJ'] == cnpj, ['REG', 'COD_PART', 'NOME', 'COD_PAIS', 'CNPJ', 'IE', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'UF', 'CEP', 'FONE']]
            fornecedor = fornecedor.iloc[[0]]
            fornecedor['COD_PAIS'] = fornecedor['COD_PAIS'].astype(int)
            fornecedor['IE'] = fornecedor['IE'].str.replace('.', '')
            fornecedor['COD_MUN'] = fornecedor['COD_MUN'].astype(int)
            fornecedor['COD_PART'] = fornecedor['CNPJ']
            fornecedor['FONE'] = fornecedor['FONE'] if pd.notnull(fornecedor['FONE'].values[0]) else '9999999999'
            fornecedor['CEP'] = fornecedor['CEP'] if pd.notnull(fornecedor['CEP'].values[0]) else '99999999'
            pd.set_option('future.no_silent_downcasting', True)
            fornecedor = fornecedor.fillna('')
            fornecedor = fornecedor[['REG','COD_PART', 'NOME', 'COD_PAIS', 'CNPJ', 'IE', 'UF', 'CEP', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'FONE']]
            fornecedor_str = '0' + '|'.join(fornecedor.astype(str).values[0])
            novos_registros.append(fornecedor_str)

    # Inserir os novos registros no arquivo .cat83
    with open(arquivo, 'r+', encoding='iso-8859-1') as f:
        linhas = f.readlines()
        posicao_insercao = 2  # Inserir na linha 3 (índice 2)
        for novo_registro in novos_registros:
            linhas.insert(posicao_insercao, novo_registro + "\n")  # Insere cada novo registro na linha 3 em diante
            posicao_insercao += 1
        f.seek(0)
        f.writelines(linhas)
        f.truncate()
        messagebox.showinfo("Sucesso", f"Registros adicionados com sucesso em {arquivo}")

def selecionar_arquivo_cat():
    global arquivo
    arquivo = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                        title="Selecionar ArquivoCat",
                                        filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))

def selecionar_base0150():
    global base0150_path
    base0150_path = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                               title="Selecionar a Base 0150",
                                               filetypes=(("Arquivos Excel", "*.xlsx"), ("todos os arquivos", "*.*")))

def selecionar_arquivos():
    global arquivo
    global base0150_path
    arquivo = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                        title="Selecionar ArquivoCat",
                                        filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))
    
    base0150_path = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                               title="Selecionar a Base 0150",
                                               filetypes=(("Arquivos Excel", "*.xlsx"), ("todos os arquivos", "*.*")))
    if arquivo and base0150_path:
        iniciar()

# Interface gráfica
janela = tk.Tk()
janela.title("Processador de Arquivos")
janela.geometry("244x150")  # 'largura' e 'altura'

# Botão para selecionar o arquivo
botao_selecionar = tk.Button(janela, text="Selecionar ArquivoCat", command=selecionar_arquivo_cat, height=2, width=20)
botao_selecionar.pack(pady=6)

# Botão para selecionar a base 0150
botao_selecionar_base = tk.Button(janela, text="Selecionar a Base 0150", command=selecionar_base0150, height=2, width=20)
botao_selecionar_base.pack(pady=6)

# Botão para iniciar o processamento
botao_iniciar = tk.Button(janela, text="Iniciar", command=lambda: [iniciar(),janela.destroy(), janela.quit()])  # Encerra a janela principal
botao_iniciar.pack(pady=6)

janela.mainloop()

# Chame o segundo código no final da função do segundo código
def segundo_codigo():
   import pandas as pd  # noqa: F401
import tkinter as tk  # noqa: E402
from tkinter import filedialog, messagebox  # noqa: E402, F811

caminho_arquivos = r'C:\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""
base0150_path = ""

def preencher_vazios():
    global arquivo
    global base0150_path

    def abrir_janela(texto):
        janela_popup = tk.Tk()
        texto_editavel = tk.Text(janela_popup)
        texto_editavel.insert(tk.END, texto)
        texto_editavel.pack()
        botao_ok = tk.Button(janela_popup, text="Confirmar", command=janela_popup.destroy)
        botao_ok.pack()
        janela_popup.mainloop()

    try:
        with open(arquivo, 'r', encoding='iso-8859-1') as f:
            linhas = f.readlines()
    except FileNotFoundError:
        abrir_janela("Arquivo não encontrado.")
        return

    # Começar a partir da linha 3 (índice 2)
    registros_0150 = [linha.strip().split('|') for linha in linhas[2:] if linha.startswith('0150')]

    # Garantir que todas as linhas tenham 14 colunas
    for registro in registros_0150:
        while len(registro) < 14:
            registro.append('')  # Preencher com vazio se faltar colunas

    tabela1 = pd.DataFrame(registros_0150, columns=['REG', 'COD_PART', 'NOME', 'COD_PAIS', 'CNPJ', 'IE', 'UF', 'CEP', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'FONE'])

    base0150 = pd.read_excel(base0150_path, dtype=str)

    novos_registros = []

    for index, row in tabela1.iterrows():
        cnpj = row['CNPJ']
        if cnpj in base0150['CNPJ'].values:
            fornecedor = base0150.loc[base0150['CNPJ'] == cnpj, ['REG', 'COD_PART', 'NOME', 'COD_PAIS', 'CNPJ', 'IE', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'UF', 'CEP', 'FONE']]
            fornecedor = fornecedor.iloc[0]
            for col in ['REG', 'COD_PART', 'NOME', 'COD_PAIS', 'IE', 'END', 'NUM', 'COMPL', 'BAIRRO', 'COD_MUN', 'UF', 'CEP', 'FONE']:
                if pd.isnull(row[col]) or row[col] == '':
                    row[col] = fornecedor[col]

        novo_registro = '|'.join(row.astype(str).values)
        # Substituir "nan" por ""
        novo_registro = novo_registro.replace('nan', '')
        novos_registros.append(novo_registro)

    with open(arquivo, 'w', encoding='iso-8859-1') as f:
    # Manter as duas primeiras linhas originais
     for linha in linhas[:2]:
        f.write(linha)

    # Escrever todos os registros, modificados ou não
     for linha in linhas[2:]:  # Começar a partir da terceira linha
        if linha.startswith('0150'):
            # Se a linha começa com 0150, usar a versão modificada
            f.write(novos_registros.pop(0) + '\n') 
        else:
            # Caso contrário, manter a linha original
            f.write(linha) 

    messagebox.showinfo("Sucesso", f"Registros atualizados com sucesso em {arquivo}")

def selecionar_arquivo_cat():
    global arquivo
    arquivo = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                        title="Selecionar ArquivoCat",
                                        filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))

def selecionar_base0150():
    global base0150_path
    base0150_path = filedialog.askopenfilename(initialdir=caminho_arquivos,
                                               title="Selecionar a Base 0150",
                                               filetypes=(("Arquivos Excel", "*.xlsx"), ("todos os arquivos", "*.*")))

# Interface gráfica
janela = tk.Tk()
janela.title("Preenchedor de Campos Vazios")
janela.geometry("244x150")

# Botão para selecionar o arquivo
botao_selecionar = tk.Button(janela, text="Selecionar ArquivoCat", command=selecionar_arquivo_cat, height=2, width=20)
botao_selecionar.pack(pady=6)

# Botão para selecionar a base 0150
botao_selecionar_base = tk.Button(janela, text="Selecionar a Base 0150", command=selecionar_base0150, height=2, width=20)
botao_selecionar_base.pack(pady=6)

# Botão para iniciar o preenchimento
botao_iniciar = tk.Button(janela, text="Iniciar", command=lambda: [preencher_vazios(), janela.destroy(), janela.quit()])
botao_iniciar.pack(pady=6)

janela.mainloop()

def terceiro_codigo():
 from tkinter import messagebox  # noqa: F401
import pandas as pd  # noqa: E402, F811
import os  # noqa: E402, F401, F811
import tkinter as tk  # noqa: E402
from tkinter import Tk, Label, Button, Text, Toplevel, END, filedialog  # noqa: E402, F401, F811

MAPA_REGISTROS = {
    '5010': 2, '5015': 13, '5060': 2, '5065': 12, '5080': 2, '5085': 12, '5100': 2,
    '5110': 2, '5150': 2, '5155': 2, '5160': 10, '5165': 2, '5170': 2, '5175': 12,
    '5180': 2, '5185': 10, '5190': 2, '5195': 9, '5210': 2, '5215': 10, '5230': 2,
    '5235': 2, '5240': 10, '5260': 2, '5265': 2, '5270': 2, '5275': 12, '5310': 2,
    '5360': 2, '5410': 2, '5550': 2, '5555': 11, '5590': 2, '5595': 2
}
CAMINHO_ARQUIVOS = r'C:\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""
base0200_path = ""  # Variável global para armazenar o caminho da base 0200
itens_info = []  # Lista global para armazenar os itens inseridos manualmente

def processar_dados():
    global arquivo
    global base0200_path
    if not arquivo or not base0200_path:  # Verifica se ambos os arquivos foram selecionados
        messagebox.showinfo("Erro", "Por favor, selecione ambos os arquivos antes de iniciar.")
        return

    # Tente ler o arquivo com várias codificações
    codificacoes = ['utf-8', 'latin-1', 'iso-8859-1']
    for cod in codificacoes:
        try:
            with open(arquivo, 'r', encoding=cod) as f:
                linhas = f.readlines()
            break  # Se a leitura do arquivo for bem-sucedida, sai do loop
        except UnicodeDecodeError:
            pass  # Se ocorrer um erro, tenta a próxima codificação
    else:
        messagebox.showinfo("Erro", "Não foi possível ler o arquivo com nenhuma das codificações.")
        return

    df_5xxx = []
    df_0200 = []
    for linha in linhas:
        campos = linha.strip().split('|')
        if campos[0] in MAPA_REGISTROS:
            df_5xxx.append(campos)
        elif campos[0] == '0200':
            df_0200.append(campos)
    df_5xxx = pd.DataFrame(df_5xxx)
    df_0200 = pd.DataFrame(df_0200)
    df_0200[1] = df_0200[1].astype(str).str.strip()

    # Leitura do arquivo Excel com COD_GEN como texto
    df_excel = pd.read_excel(
        base0200_path,
        usecols=['REG', 'COD_ITEM', 'DESCR_ITEM', 'UNI', 'COD_GEN'],
        dtype={'COD_GEN': str}
    )
    df_excel = df_excel.astype(str)
    df_excel['COD_ITEM'] = df_excel['COD_ITEM'].str.strip()

    codigos_itens = {}
    for registro, coluna in MAPA_REGISTROS.items():
        df_registro = df_5xxx[df_5xxx[0] == registro]
        codigos_itens[registro] = df_registro.iloc[:, coluna - 1].astype(str).str.strip().unique()
    codigos_0200 = df_0200[1].unique()
    itens_faltantes = set()
    for registro, codigos in codigos_itens.items():
        for codigo in codigos:
            if codigo not in codigos_0200:
                itens_faltantes.add(codigo)
    info_item = pd.DataFrame()  # Inicializa info_item como um DataFrame vazio
    for item in itens_faltantes:
        info_item = df_excel[df_excel['COD_ITEM'] == item]
        if not info_item.empty:
            info_item = info_item.iloc[0]  # Pega a primeira linha se encontrar o item
            cod_gen = info_item.get('COD_GEN', '')  # Obtém COD_GEN ou vazio se não existir

            # Verifica se COD_GEN é válido (não nulo, não NaN)
            if pd.notnull(cod_gen) and not pd.isna(cod_gen) and cod_gen.lower() != 'nan':
                cod_gen_str = f"|{cod_gen}"
            else:
                cod_gen_str = "|"
            # Gera a string com a formatação correta (sem alterações)
            info_str = f"0200|{info_item.get('COD_ITEM', '')}|{info_item.get('DESCR_ITEM', '')}|{info_item.get('UNI', '')}{cod_gen_str}"
            itens_info.append(info_str)
        else:
            abrir_janela(f"Item {item} não encontrado na base 0200. Por favor, insira os detalhes manualmente:")

    # Inserção e escrita no arquivo
    novas_linhas = []
    inseriu_info = False
    for i, linha in enumerate(linhas):
        if linha.startswith('0150'):
            if not inseriu_info and i + 1 < len(linhas) and not linhas[i + 1].startswith('0150'):
                novas_linhas.append(linha.rstrip() + "\n")  # Remove espaços em branco e adiciona quebra de linha
                for info_str in itens_info:
                    novas_linhas.append(info_str + "\n")  # Insere cada informação em uma linha separada com quebra de linha
                inseriu_info = True
            else:
                novas_linhas.append(linha.rstrip() + "\n")  # Remove espaços em branco e adiciona quebra de linha
        else:
            novas_linhas.append(linha.rstrip() + "\n")  # Remove espaços em branco e adiciona quebra de linha
    with open(arquivo, 'w', encoding='utf-8') as f:
        f.writelines(novas_linhas)
    # Mensagem final
    messagebox.showinfo("Sucesso", "Informações inseridas no arquivo com sucesso.")
    root.quit()  # Encerra a execução do programa

def abrir_janela(texto):
    janela_popup = tk.Toplevel()
    janela_popup.title("Adicionar Item")
    
    label = tk.Label(janela_popup, text=texto)
    label.pack()
    
    entrada_texto = tk.Text(janela_popup, height=10, width=50)
    entrada_texto.pack()
    
    def inserir_item():
        item_texto = entrada_texto.get("1.0", END).strip()
        if item_texto:  # Verifica se o item foi inserido corretamente
            itens_info.append(item_texto)
            entrada_texto.delete("1.0", END)
            janela_popup.destroy()  # Fecha a janela após a inserção correta
            root.quit()  # Encerra a execução do programa
    
    botao_inserir = tk.Button(janela_popup, text="Inserir Item 0200", command=inserir_item)
    botao_inserir.pack()
    
    janela_popup.mainloop()

def selecionar_arquivo_cat():
    global arquivo
    arquivo = filedialog.askopenfilename(initialdir=CAMINHO_ARQUIVOS,
                                         title="Selecionar ArquivoCat",
                                         filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))

def selecionar_base0200():
    global base0200_path
    base0200_path = filedialog.askopenfilename(initialdir=CAMINHO_ARQUIVOS,
                                               title="Selecionar Base do 0200",
                                               filetypes=(("Arquivos Excel", "*.xlsx"), ("todos os arquivos", "*.*")))

root = tk.Tk()
root.title("Processador de Arquivos")
root.geometry("244x150")
botao_selecionar = Button(root, text="Selecionar ArquivoCat", command=selecionar_arquivo_cat, height=2, width=20)
botao_selecionar.pack(pady=6)
botao_selecionar_base = Button(root, text="Selecionar Base do 0200", command=selecionar_base0200, height=2, width=20)
botao_selecionar_base.pack(pady=6)
botao_iniciar = Button(root, text="Iniciar", command=processar_dados)
botao_iniciar.pack(pady=6)
root.mainloop()

# Chame o quarto código no final da função do terceiro código
def quarto_codigo():
 from tkinter import filedialog, messagebox  # noqa: F401
import pandas as pd  # noqa: E402, F811
import os  # noqa: E402, F811
import tkinter as tk  # noqa: E402
from tkinter import Tk, Label, Button, scrolledtext, Toplevel, END, filedialog  # noqa: E402, F401, F811

MAPA_REGISTROS = {
    '5010': 2, '5015': 13, '5060': 2, '5065': 12, '5080': 2, '5085': 12, '5100': 2,
    '5110': 2, '5150': 2, '5155': 2, '5160': 10, '5165': 2, '5170': 2, '5175': 12,
    '5180': 2, '5185': 10, '5190': 2, '5195': 9, '5210': 2, '5215': 10, '5230': 2,
    '5235': 2, '5240': 10, '5260': 2, '5265': 2, '5270': 2, '5275': 12, '5310': 2,
    '5360': 2, '5410': 2, '5550': 2, '5555': 11, '5590': 2, '5595': 2
}
CAMINHO_ARQUIVOS = r'C:\Users\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""

def abrir_janela(itens_info, remover=True):  # noqa: F811
    janela = Toplevel(root)
    janela.title("Informações dos Itens")
    texto = scrolledtext.ScrolledText(janela, wrap=tk.WORD, font=("Arial", 12))
    texto.pack(expand=True, fill="both", padx=20, pady=20)
    for info in itens_info:
        texto.insert(END, info + "\n")
    if remover:
        btn_acao = tk.Button(janela, text="Remover Itens", command=lambda: remover_itens(itens_info, janela))
    else:
        btn_acao = tk.Button(janela, text="Confirmar", command=root.destroy)
    btn_acao.pack()

def remover_itens(itens_info, janela_anterior):
    global arquivo
    if not arquivo:
        abrir_janela(["Nenhum arquivo selecionado."])
        return
    codigos_a_remover = {info.split('|')[1] for info in itens_info}
    with open(os.path.join(CAMINHO_ARQUIVOS, arquivo), 'r', encoding='iso-8859-1') as f:
        linhas_originais = f.readlines()
    with open(os.path.join(CAMINHO_ARQUIVOS, arquivo), 'w', encoding='iso-8859-1') as f:
        for linha in linhas_originais:
            campos = linha.strip().split('|')
            if campos[0] == '0200' and campos[1] in codigos_a_remover:
                continue
            f.write(linha)
    janela_anterior.destroy()
    messagebox.showinfo("Sucesso", "Itens removidos com sucesso.")
    root.destroy()

def processar_dados():
    global arquivo
    if not arquivo:
        abrir_janela(["Nenhum arquivo selecionado."])
        return
    codificacoes = ['utf-8', 'latin-1', 'iso-8859-1']
    for cod in codificacoes:
        try:
            with open(os.path.join(CAMINHO_ARQUIVOS, arquivo), 'r', encoding=cod) as f:
                linhas = f.readlines()
            break
        except UnicodeDecodeError:
            pass
    else:
        messagebox.showinfo("Erro", "Não foi possível ler o arquivo com nenhuma das codificações.")
        return
    df_5xxx = []
    df_0200 = []
    for linha in linhas:
        campos = linha.strip().split('|')
        if campos[0] in MAPA_REGISTROS:
            df_5xxx.append(campos)
        elif campos[0] == '0200':
            df_0200.append(campos)
    df_5xxx = pd.DataFrame(df_5xxx)
    df_0200 = pd.DataFrame(df_0200)
    df_0200[1] = df_0200[1].astype(str).str.strip().str.upper()
    codigos_itens = {}
    for registro, coluna in MAPA_REGISTROS.items():
        df_registro = df_5xxx[df_5xxx[0] == registro]
        codigos_itens[registro] = df_registro.iloc[:, coluna - 1].astype(str).str.strip().str.upper().unique()
    codigos_0200 = df_0200[1].unique()
    itens_faltantes = set()
    for codigo in codigos_0200:
        if not any(codigo in codigos for codigos in codigos_itens.values()):
            itens_faltantes.add(codigo)
    itens_info = []
    for item in itens_faltantes:
        info_item = df_0200[df_0200[1] == item]
        if not info_item.empty:
            info_item = info_item.iloc[0]
            info_str = '|'.join(['0200', info_item[1], info_item[2], info_item[3], info_item[4]])
            itens_info.append(info_str)
    abrir_janela(itens_info)

def selecionar_arquivo():
    global arquivo
    arquivo = filedialog.askopenfilename(
        initialdir=CAMINHO_ARQUIVOS,
        title="Selecione o ArquivoCat",  
        filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*"))
    )

root = tk.Tk()
root.title("Processador de Arquivos")
root.geometry("244x120")
botao_selecionar = Button(root, text="Selecionar ArquivoCat", command=selecionar_arquivo, height=2, width=20)
botao_selecionar.pack(pady=15)
botao_iniciar = Button(root, text="Iniciar", command=processar_dados)
botao_iniciar.pack(pady=10)
root.mainloop()

# Chame o Quinto código no final da função do terceiro código
def quinto_codigo():
 from tkinter import messagebox  # noqa: F401
import pandas as pd  # noqa: E402, F811
import tkinter as tk  # noqa: E402
from tkinter import filedialog  # noqa: E402, F811

arquivo_txt = None
arquivo_xlsx = None

def selecionar_arquivo_cat():
    global arquivo_txt
    arquivo_txt = filedialog.askopenfilename(filetypes=[("Arquivos TXT", "*.txt")], title="Selecionar ArquivoCat")

def selecionar_arquivo_historico():
    global arquivo_xlsx
    arquivo_xlsx = filedialog.askopenfilename(filetypes=[("Arquivos Excel", "*.xlsx")], title="Selecionar Histórico 0200")

def iniciar_comparacao():
    if arquivo_txt and arquivo_xlsx:
        comparar_arquivos(arquivo_txt, arquivo_xlsx)
    else:
        print("Selecione ambos os arquivos antes de iniciar.")

def encerrar_janela():  # Definindo a função encerrar_janela() no escopo global
    janela_encerrar = tk.Toplevel(root)
    janela_encerrar.title("Encerrar")
    tk.Label(janela_encerrar, text="Comparação concluída com sucesso!").pack()
    tk.Button(janela_encerrar, text="Encerrar", command=root.destroy).pack()  # Encerra o programa quando clicado

def ler_arquivo_com_codificacao(arquivo_txt, codificacoes):
    for cod in codificacoes:
        try:
            with open(arquivo_txt, 'r', encoding=cod) as f:
                lines = f.readlines()
                print(f"Arquivo lido com sucesso usando a codificação {cod}!")
                return lines
        except UnicodeDecodeError:
            continue
    return None

def comparar_arquivos(arquivo_txt, arquivo_xlsx):
    # Carrega todas as abas da planilha Excel em um dicionário
    xls = pd.ExcelFile(arquivo_xlsx)
    abas = xls.sheet_names

    # Dicionário para armazenar os dados de todas as abas
    dict_excel = {}

    # Itera sobre cada aba (período)
    for aba in abas:  # Corrigido aqui
        df_excel = pd.read_excel(arquivo_xlsx, sheet_name=aba, usecols=['COD_ITEM', 'DESCR_ITEM'], dtype=str)

        # Atualiza o dicionário com os dados da aba atual, preservando maiúsculas e minúsculas
        dict_excel.update(df_excel.set_index('COD_ITEM')['DESCR_ITEM'].to_dict())

    # Abre o arquivo TXT para leitura e escrita
    codificacoes = ['utf-8', 'latin1', 'iso-8859-1']  # Adicione aqui as codificações que você deseja tentar
    lines = ler_arquivo_com_codificacao(arquivo_txt, codificacoes)
    if lines is None:
        print("Não foi possível ler o arquivo com nenhuma das codificações fornecidas.")
        return

    with open(arquivo_txt, 'w', encoding='utf-8') as f:        
        f.seek(0)  # Move o cursor para o início do arquivo

        # Processa cada linha do arquivo TXT
        for line in lines:
            columns = line.split('|')

            # Verifica se a linha corresponde a um item (código 0200)
            if columns[0].strip() == '0200':
                item_code = columns[1].strip()
                item_description = columns[2].strip()

                # Verifica se o código do item existe na planilha
                if item_code in dict_excel:
                    # Compara as descrições, preservando maiúsculas e minúsculas
                    if item_description != dict_excel[item_code]:
                        columns[2] = dict_excel[item_code]
                        line = '|'.join(columns)

            f.write(line)

        f.truncate()  # Remove qualquer conteúdo extra no final do arquivo

    messagebox.showinfo("Sucesso","Comparação concluída com sucesso!")
    encerrar_janela()  # Adicione esta linha

root = tk.Tk()
root.title("Processador de Arquivos")
root.geometry("244x150")

botao_selecionar = tk.Button(root, text="Selecionar ArquivoCat", command=selecionar_arquivo_cat, height=2, width=20)
botao_selecionar.pack(pady=6)

botao_selecionar_base = tk.Button(root, text="Selecionar Histórico 0200", command=selecionar_arquivo_historico, height=2, width=20)
botao_selecionar_base.pack(pady=6)

botao_iniciar = tk.Button(root, text="Iniciar", command=iniciar_comparacao)
botao_iniciar.pack(pady=6)

root.mainloop()

def sexto_codigo():
 import os  # noqa: F401
import tkinter as tk  # noqa: E402
from tkinter import filedialog, messagebox  # noqa: E402, F811

CAMINHO_ARQUIVOS = r'C:\Users\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""

def ler_arquivo_com_codificacao(arquivo, codificacoes):  # noqa: F811
    for cod in codificacoes:
        try:
            with open(arquivo, 'r', encoding=cod) as f:
                lines = f.readlines()
                print(f"Arquivo lido com sucesso usando a codificação {cod}!")
                return lines
        except UnicodeDecodeError:
            continue
    return None

def processar_arquivo():
    global arquivo
    if not arquivo:
        messagebox.showinfo("Erro", "Por favor, selecione o arquivo antes de iniciar.")
        return

    # Tente ler o arquivo com várias codificações
    codificacoes = ['utf-8', 'latin1', 'iso-8859-1']
    linhas = ler_arquivo_com_codificacao(os.path.join(CAMINHO_ARQUIVOS, arquivo), codificacoes)
    if linhas is None:
        print("Não foi possível ler o arquivo com nenhuma das codificações fornecidas.")
        return

    linhas_atualizadas = []
    for i, linha in enumerate(linhas):
        campos = linha.strip().split('|')
        if campos[0] == '5085' and campos[9] == '137017':
            campos[6] = campos[1]
            if i + 1 < len(linhas):
                campos_proxima_linha = linhas[i + 1].strip().split('|')
                campos[8] = campos_proxima_linha[8]
            print('Linha alterada:', '|'.join(campos))
        linhas_atualizadas.append('|'.join(campos))

    with open(os.path.join(CAMINHO_ARQUIVOS, arquivo), 'w', encoding='utf-8') as f:
        for linha in linhas_atualizadas:
            f.write(linha + '\n')

    messagebox.showinfo("Sucesso", "Linha Estorno de Energia 137017, Corrigida!")

def selecionar_arquivo():
    global arquivo
    arquivo = filedialog.askopenfilename(initialdir=CAMINHO_ARQUIVOS,
                                          title="Selecionar ArquivoCat",
                                          filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))

janela = tk.Tk()
janela.title("Processador de Arquivos")
janela.geometry("244x150")

botao_selecionar = tk.Button(janela, text="Selecionar ArquivoCat", command=selecionar_arquivo, height=2, width=20)
botao_selecionar.pack(pady=6)

botao_processar = tk.Button(janela, text="Processar Arquivo", command=processar_arquivo, height=2, width=20)
botao_processar.pack(pady=6)

botao_finalizar = tk.Button(janela, text="Finalizar Tarefa!", command=janela.destroy)
botao_finalizar.pack(pady=6)

janela.mainloop()

def setimo_codigo():
 import os  # noqa: F401
import tkinter as tk  # noqa: E402
from tkinter import filedialog, messagebox  # noqa: E402, F811

CAMINHO_ARQUIVOS = r'C:\Users\Área de Trabalho\piloto.cat83\arquivoscat'
arquivo = ""

def ler_arquivo_com_codificacao(arquivo, codificacoes):  # noqa: F811
    for cod in codificacoes:
        try:
            with open(arquivo, 'r', encoding=cod) as f:
                lines = f.readlines()
                print(f"Arquivo lido com sucesso usando a codificação {cod}!")
                return lines
        except UnicodeDecodeError:
            continue
    return None

def processar_arquivo():
    global arquivo
    if not arquivo:
        messagebox.showinfo("Erro", "Por favor, selecione o arquivo antes de iniciar.")
        return

    # Tente ler o arquivo com várias codificações
    codificacoes = ['utf-8', 'latin1', 'iso-8859-1']
    linhas = ler_arquivo_com_codificacao(os.path.join(CAMINHO_ARQUIVOS, arquivo), codificacoes)
    if linhas is None:
        print("Não foi possível ler o arquivo com nenhuma das codificações fornecidas.")
        return

    # Obtenha o mês e o ano da primeira linha
    month_year = linhas[0].split('|')[4]

    # Prepare a data a ser inserida
    date_to_insert = '01' + month_year

    # Processe as linhas
    for i in range(len(linhas)):
        fields = linhas[i].split('|')
        if (fields[0] == '5160' or fields[0] == '5315') and fields[2] == '':
            fields[2] = date_to_insert
            linhas[i] = '|'.join(fields)

    # Escreva as linhas processadas de volta no arquivo
    with open(os.path.join(CAMINHO_ARQUIVOS, arquivo), 'w', encoding='utf-8') as f:
        f.writelines(linhas)

    messagebox.showinfo("Sucesso", "O arquivo foi processado com sucesso!")

def selecionar_arquivo():
    global arquivo
    arquivo = filedialog.askopenfilename(initialdir=CAMINHO_ARQUIVOS,
                                          title="Selecionar ArquivoCat",
                                          filetypes=(("Arquivos de texto", "*.txt"), ("todos os arquivos", "*.*")))

janela = tk.Tk()
janela.title("Processador de Arquivos")
janela.geometry("244x150")

botao_selecionar = tk.Button(janela, text="Selecionar ArquivoCat", command=selecionar_arquivo, height=2, width=20)
botao_selecionar.pack(pady=6)

botao_processar = tk.Button(janela, text="Processar Arquivo", command=processar_arquivo, height=2, width=20)
botao_processar.pack(pady=6)

botao_finalizar = tk.Button(janela, text="Finalizar Tarefa!", command=janela.destroy)
botao_finalizar.pack(pady=6)

janela.mainloop()

def iniciar():  # noqa: F811
    global arquivo_global
# Chame os códigos na ordem desejada
primeiro_codigo()
segundo_codigo()
terceiro_codigo()
quarto_codigo()
quinto_codigo()
sexto_codigo()
setimo_codigo()
