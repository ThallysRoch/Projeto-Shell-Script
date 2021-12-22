#Dupla com:
#Pedro Henrique Firmino Ceruti - 20201380023
#pedro.ceruti@academico.ifpb.edu.br
#
#

#!/bin/bash
KEY=$RANDOM


filtro_caminho="/root/teste/projeto/.filtro.txt"
temporizador_caminho="/root/teste/projeto/.temporizador.txt"
matar_caminho="/root/teste/projeto/.matadouro.txt"


echo "default" > /root/teste/projeto/.filtro.txt
echo "5" > /root/teste/projeto/.temporizador.txt
echo "" > /root/teste/projeto/.matadouro.txt

function filtrar (){
	filtro=$(yad --title="Informe o Filtro" --form --center --width=300 --field Filtro: --button="Confirmar":0 --button="Cancelar":1)
	echo ${filtro:0:-1} > /root/teste/projeto/.filtro.txt
}

export -f filtrar

function matar (){
	processo_matar=$(yad --title="Finalizar Processo" --center --form --field PID --button"OK:0" --button"Cancelar:1" --width=500)
	echo "$processo_matar" | sed -e 's/|//g' > /root/teste/projeto/.matadouro.txt
	kill -9 $(cat /root/teste/projeto/.matadouro.txt)
}
export -f matar

function limpar_filtros (){
	echo "default" > /root/teste/projeto/.filtro.txt
}

export -f limpar_filtros

function init_process(){
    local process_to_init=$(yad --title="Iniciar novo processo" --center --form --field="Nome do processo/programa: " --button="OK:0" --button="Cancelar:1" --width=512)
    ${process_to_init:0:-1} &
}

export -f init_process

yad --plug=$KEY --list --listen --tabnum=1 \
	--column=$"PID" --column="Usuário" \
	--column=$"CPU %" --column=$"Memória %" --column=$"Processos" \ < <(
while true;do
	filtro_atual=$(echo "$(< $filtro_caminho)")

	if [[ "$filtro_atual" == "default" ]]; then
		awk '( NR>1 ) {printf "%s\n%s\n%s\n%s\n%s\n",$2,$1,$3,$4,$11}' <(ps -aux --sort=-pcpu) > .temp.txt
    
	elif [[ "$filtro_atual" != "default" ]]; then
		awk '( NR>0 ) {printf "%s\n%s\n%s\n%s\n%s\n",$2,$1,$3,$4,$11}' <(grep "$filtro_atual" <(ps -aux --sort=-pcpu))| head -n -5 > .temp.txt
    
	fi

	echo "$(< .temp.txt)"
	echo "" > .temp.txt

	sleep $(cat ${temporizador_caminho})
	echo -e '\f'
done ) &


yad --notebook --key=$KEY --stack --center --title="Gerenciador" \
  --button="Filtrar":"bash -c filtrar" \
	--button="Matar Processo":"bash -c matar" \
	--button="Limpar Filtros":"bash -c limpar_filtros" \
  --button="Iniciar Processo":"bash -c init_process" \
  --width=512 --height=500 --key=$KEY --tab=$"Processos" \
  --active-tab=${1:-1}
