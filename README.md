# ShellMikrotik
SSH +ShellScript to manage Mikrotik equipments

#!/bin/bash

data=$(date +%d-%m-%Y)

mkdir -p /home/deise/Sistema/Backup/rbs/$data
mkdir -p /home/deise/Sistema/Lista-Equipamentos/rbs

touch /home/deise/Sistema/Lista-Equipamentos//rbs/rbs.txt
touch /home/deise/Sistema/Lista-Equipamentos/rbs/rbteste.txt
chmod 777 -R /home/deise/Sistema/*

#########################################################################################################################################################
################################################### Area destinada as funcoes do programa ###############################################################
#########################################################################################################################################################

#########################################################################################################################################################
##############################################################Funcao de controle#########################################################################
#########################################################################################################################################################

func_main(){

clear

data=$(date +%d'/'%m'/'%Y)

echo "Bem vindo " - $data

echo "#####-MENU PRINCIPAL-#####"
echo ""
echo "[ 1 ] - Adicionar um novo Host"
echo "[ 2 ] - Adicionar usuario"
echo "[ 3 ] - Backup de Hosts"
echo "[ 4 ] - Comandos individuais"
echo "[ 0 ] - SAIR"
read principal

case $principal in

1)

func_adiciona_host

;;

2)

func_adc_user

;;

3)

func_backup

;;

4)

func_comando_solo

;;

0)
clear; 
echo "" ;
echo "Finalizando Programa" ;
sleep 1;
clear;
;;

esac
}

func_adiciona_host(){

clear

echo "##-FUNCAO ADICIONAR HOST-##"
echo "[ 1 ] - Adicionar Host"
echo "[ 0 ] - VOLTAR"
read destino

case $destino in

1)
clear
echo "Digite o HOST que deseja adicionar"
read host

##Adiciona o host na lista##
echo "$host" >> /home/deise/Sistema/Lista-Equipamentos/rbs/rbs.txt

echo "Host adicionado!"

echo "Aperte qualquer botao para voltar ao menu anterior"
read
func_adiciona_host
;;

0)

func_main

;;
esac

}

func_adc_user(){
clear
echo "#####-MENU ADC USER-#####"
echo ""
echo "[ 1 ] - Adicionar usuario a todos Hosts"
echo "[ 2 ] - Adicionar usuario a um ou mais hosts"
echo "[ 0 ] - Voltar"
read adicionar

case $adicionar in

1)
echo "Digite o usuario que deseja adicionar"
read usuarionovo
echo "Digite a senha que deseja "
read passwordnovo
echo "Digite um usuario existente para criar o cadastro"
read userold
for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/rbs.txt); do
ssh -p 8462 $userold@$i /user add name=$usuarionovo password=$passwordnovo group=full

echo "Utilize o comando put id_dsa.pub"
sftp -P 8462 $usuarionovo@$i

ssh -p 8462 $usuarionovo@$i /user ssh-keys import public-key-file=id_dsa.pub user=$usuarionovo

echo "Usuario $usuarionovo adicionado a $i com sucesso"
done
echo "Aperte qualquer botao para voltar ao menu anterior"
read
func_adc_user
;;

2)
echo "Quantos hosts receberao o comando?"
read quantidade
echo "" > /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt

for ((x=0; x < $quantidade; x++));do
echo "Digite a RB que deseja executar o comando"
read routerboard
echo "$routerboard" >> /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt
done

echo "Digite o usuario que deseja adicionar"
read usuarionovo
echo "Digite a senha que deseja "
read passwordnovo
echo "Digite um usuario existente para criar o cadastro"
read userold

for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt); do
ssh -p 8462 $userold@$i /user add name=$usuarionovo password=$passwordnovo group=full 

echo "Utilize o comando put id_dsa.pub"
sftp -P 8462 $usuarionovo@$i

##Associa a chave SSH ao usuario##
ssh -p 8462 $usuarionovo@$i /user ssh-keys import public-key-file=id_dsa.pub user=$usuarionovo
echo "Usuario $usuarionovo adicionado a $i com sucesso"
done 

echo "Aperte qualquer botao para voltar ao menu anterior"
read
func_adc_user
;;

0)
func_main

;;
esac
}

dia=$(date +%d-%m-%Y)

func_backup(){

clear

echo "BACKUP de Equipamentos da Lista"
echo ""
echo "[ 1 ] - Iniciar Backup"
echo "[ 2 ] - VOLTAR"
read opcao

case $opcao in

1)
for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/rbs.txt); do
ssh -p 8462 backup@$i /system backup save name=$i.backup dont-encrypt=yes
done

for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/rbs.txt); do
sftp -P 8462 backup@$i:$i.backup
done

echo "$dia"
mv /home/deise/Sistema/*.backup /home/deise/Sistema/Backup/rbs/$dia

echo "O BACKUP DAS TRASMISSORAS FOI GERADO E SALVO NA PASTA $dia"

##comando para pausar o sistema##
echo "Aperte qualquer botao para voltar ao menu anterior"
read

##Retorna ao menu de opçoes##
func_backup

;;

2)
func_main

;;
esac

}

func_comando_solo(){

clear
echo "#####-MENU COMANDOS GERAIS-#####"
echo ""
echo "[ 1 ] - Inserir comando para todos Hosts cadastrados"
echo "[ 2 ] - Inserir comando a Host especifico ou mais de um host fora da lista"
echo "[ 0 ] - Voltar"
read comm

case $comm in

1)
echo "Digite o comando desejado"
read comando

for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/rbs.txt); do

ssh -p 8462 backup@$i $comando
echo "Comando $comando realizado com sucesso em $i "
done

##comando para pausar o sistema##
echo "Aperte qualquer botao para voltar ao menu anterior"
read

##Retorna ao menu de opçoes##
func_main
;;

2)
echo "Quantos hosts receberao o comando?"
read quantidade
echo "" > /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt

for ((x=0; x < $quantidade; x++));do
echo "Digite a RB que deseja executar o comando"
read routerboard
echo "$routerboard" >> /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt
done

echo "Digite o comando desejado"
read comando2

for i in $(cat /home/deise/Sistema/Lista-Equipamentos/rbs/temporario.txt); do
ssh -p 8462 backup@$routerboard $comando2
echo "Comando $comando realizado com sucesso em $i "
done

echo "Aperte qualquer botao para voltar ao menu anterior"
read

Retorna ao menu de opçoes#
func_comando_solo
;;

0)
func_main

;;
esac
}

#########################################################################################################################################################
############################################################# Corpo principal do programa ###############################################################
#########################################################################################################################################################

clear

data=$(date +%d'/'%m'/'%Y)


echo "Bem vindo " - $data

echo "#####-MENU PRINCIPAL-#####"
echo ""
echo "[ 1 ] - Adicionar um novo Host"
echo "[ 2 ] - Adicionar usuario"
echo "[ 3 ] - Backup de Hosts"
echo "[ 4 ] - Comandos individuais"
echo "[ 0 ] - SAIR"
read principal

case $principal in

1)

func_adiciona_host

;;

2)

func_adc_user

;;

3)

func_backup

;;

4)

func_comando_solo

;;

0)
clear; 
echo "" ;
echo "Finalizando Programa" ;
sleep 1;
clear;
;;

esac
