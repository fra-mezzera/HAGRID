MODULE 01 FILE DESCRIPTION - INPUTS TIME USE SURVEYS


In this folder two type of data are included:

	- in "ISTAT original TUS data" subfolder all the original dataset provided by the Italian National Institute of Statistics (ISTAT) are reported, as can be downloaded by

	https://www.istat.it/microdati/multiscopo-sulle-famiglie-uso-del-tempo-2/


	- "descrizione_dati_TUS.xlsx" --> the Excel legend file in which every Italian category is grouped and described, to properly elaborate and create synthetic individual activity profile timeseries. This file contains Italian definition since it is directly derived from the original TUS dataset, based on 2013 Italian TUS data

	- all the remaining .pkl files are derived from the initial dataset. These elaborations were required to link the identified individual class to each single timeserie, since the original dataset is anonymized due to privacy reasons. The elaborations then link the demographic information (e.g., role within the household, head of household, child; the household type, if single-person, couple with Children) and geographical and time dependent (i.e., associated trimester) to each activity profile, in order to enable the possibility to create synthetic activity profiles through a Markov-Chain method approach