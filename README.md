Bussinees Inteligence: Projeto de Dashboard para Football Analytics no Power BI

 1.Conectar as fontes de dados

Tabela de clubes.csv

 2. Ajustar, padronizar e  limpar os dados

Deve-se deixar a tabela está na mesma tipagem de valores por isso vamos selecionar em "transformar os dados” para verificar.

O mesmo procedimento foi realizado para a tabela aparicoes.csv, competicoes.csv, jogadores.csv, jogadores_tratados.csv, valores_de_jogadores.csv

Na tabela jogadores_tratados adicionamos uma coluna que concatena o game_id com o club_id para criar uma coluna de dados únicos.

Concatenado = [game_id]&"-"&[club_id]

 3. Estabelecendo as relações entre as tabelas

Precisamos estabelecer as relações de cardinalidade para estabelecer os filtros dos dados.


Tipo de relação entre tabelas: 
• Um para muitos (1,*) 
• Muitos para um (*,1) 
• Um para um (1,1) 
• Muitos para muitos (*,*)

Exemplo: Competion_ID é uma chave única pois temos apenas umas competição, Clubes é uma chave única pois os clubes não se repetem….

Não temos como relacionar a tabela de jogadores_tratadores.csv com a de aparicoes.csv devido não ter uma coluna para interligar os dados. 

Na tabela jogadores_tratados adicionamos então uma coluna que concatena o game_id com o club_id para criar uma coluna de dados únicos.

Concatenado = [game_id]&"-"&[club_id]

O mesmo procedimento foi realizado para a tabela aparicoes.csv

Concatenado = [game_id]&"-"&[player_club_id]

e assim obtivemos uma coluna igual para criar uma relação.

Criamos uma tabela calendário para servir de apoio:

calendario = CALENDAR("2012-01-01",TODAY())
Ajustando ela temos:

Calendario = ADDCOLUMNS(
CALENDAR("2012-01-01",Today()),
"Ano", YEAR([Date]),
            "Mês Num", MONTH([Date]),
            "Trim", FORMAT([Date], "q"),
            "Semana Ano", WEEKNUM([Date]),
            "Dia Semana", WEEKDAY([Date]),
            "Mês", FORMAT([Date], "mmm"),
            "Mês Completo", FORMAT([Date], "mmmm"),
            "Mês_Ano", FORMAT(MONTH([Date]), "00") & "/" & YEAR([Date]),
            "Mês-Ano", UPPER(FORMAT(MONTH([Date]), "mmm")) & "/" & YEAR([Date]),
            "Trim_Ano", FORMAT([Date], "q") & "T" & YEAR([Date]),
            "Dia_Semana", FORMAT([Date], "ddd"))

 4. Dashboard do Clube

Agora podemos começar a aplicar as informações necessárias:

Filtros:
- Clube
- Campeonatos
- Jogou em 
- Temporada

Visual:
- Tabela de resultados em gols
- Resultados do Clube 
- Aproveitamento dentro e fora de campo
- Cartão com resultado de gols dentro e fora de casa
- Aproveitamento por competição
- Aproveitamento ao longo do tempo com nível hierárquico

Adicionando uma coluna de vitórias, empate e perdas.

Resultado = IF([goals]=[foe_goals],"empate",IF([goals]<[foe_goals],"venceu","perdeu"))

Para sabermos quem foi o adversário dos jogos também adicionamos uma coluna a mais:
 
adversario = LOOKUPVALUE(clubes[pretty_name],[club_id],[foe_club_id])

Criamos uma medida nova chamada aproveitamento, sua ideia consiste em saber o número de vitórias dividido pelo número total de jogos:

Aproveitamento = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu"))/COUNTROWS(jogos_tratados)
 
Criamos também uma medida para saber a quantidade total de jogos:

quantidade_de_jogos = COUNTROWS(jogos_tratados)
 
Seria interessante saber o aproveitamento de jogos em casa, para isso precisamos adicionar um filtro no aproveitamento:
 
aproveitamento_em_casa = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu" && [played_in]="home"))/CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[played_in]="home"))
 
Da mesma maneira podemos calcular o aproveitamento fora de casa:
aproveitamento_fora_de_casa = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu" && [played_in]="away"))/CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[played_in]="away"))
 
Formatação de cores gradientes para os gráficos em radar:
Verde: #3D6131
Amarelo: #f1c40f
Vermelho: #A1343C

 5. Medidas

Foram adicionadas algumas medidas para visualização dos dados

vitorias = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu"))
derrotas = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="perdeu"))
empate = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="empate"))

quantidade_de_jogos = COUNTROWS(jogos_tratados)
Media_de_gols = AVERAGE(jogos_tratados[goals])

aproveitamento = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu"))/COUNTROWS(jogos_tratados)

aproveitamento_em_casa = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu" && [played_in]="home"))/CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[played_in]="home"))

aproveitamento_fora_de_casa = CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[resultado]="venceu" && [played_in]="away"))/CALCULATE(COUNTROWS(jogos_tratados),FILTER('jogos_tratados',[played_in]="away"))

