#include <stdio.h>
#include <stdlib.h>
#include <conio.h>

#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "semphr.h"

#include<stdio.h>
#include<stdlib.h>


static int count_id = 1;
static int flag_rek = 0;

struct{

	int id;
	char title[16];
	char type[1];
	int predecesors_id[6];
	int succesors_id[6];

}Jobs[64];

static int add_job(char name[], char type[]){

	if(strlen(name) > 16){

		printf("Greska, preveliko ime!\n");
		fflush(stdout);
		return 0;

	}else if((strcmp(type, "A") && strcmp(type, "B") && strcmp(type, "C") && strcmp(type, "D")) == 1){

		printf("Greska, pogresan tip!\n");
		fflush(stdout);
		return 0;

	}else if(name == NULL || type == NULL){

		printf("Greska, niste naveli oba karaktera!\n");
		fflush(stdout);
		return 0;

	}

	if(0){

		printf("Greska, sistem je pun!\n");
		fflush(stdout);
		return 0;

	}else{

		strcpy( Jobs[count_id].title, name);
		strcpy( Jobs[count_id].type, type);

		Jobs[count_id].id = count_id;
		count_id +=1;
		//xQueueSend(xMyQueue, &job, (TickType_t) portMAX_DELAY);

		return 1; // NAPRAVI PRINTF NAKON!!!

	}

}

int rek_ciklicni_preth(int* p_id, int* s_id){ // ne mogu da stopiram loop bez static variable
	int i;

	for(i = 0; i<5; i++){

		if(Jobs[*p_id].predecesors_id[i] == 0){
			continue;
		}
		printf("\n----------i check: %d ", Jobs[*p_id].predecesors_id[i]);
		printf("is it equal to: %d ----------\n\n",Jobs[*s_id].id);
		fflush(stdout);

		if(Jobs[*p_id].predecesors_id[i] == Jobs[*s_id].id){
			flag_rek += 1;
			printf("JEDNAKO JE---------------");
			fflush(stdout);
		}

		printf("----flag === %d \n\n", flag_rek);

		rek_ciklicni_preth(&Jobs[*p_id].predecesors_id[i], s_id);


	}

	if(flag_rek == 1){
		return 1;
	}
	return 0;
}

static void connect_jobs  (int* p_id, int* s_id){

	if(((*p_id && *s_id) == 0) || (*p_id == *s_id)){
		printf("Greska, niste validno uneli parametre! \n");
		return;
	}

	int flag = 1;
	for(int i =0; i<count_id;i++){
		if(Jobs[i].id == *p_id){
			flag = 0;
		}
	}
	if(flag == 1){
		printf("Greska, ne postoji p_id! \n");
		return;
	}
	flag = 1;
		for(int i =0; i<count_id;i++){
			if(Jobs[i].id == *s_id){
				flag = 0;
			}
		}
		if(flag == 1){
			printf("Greska, ne postoji s_id! \n");
			return;
		}

		int pos_sl;	// pozicija u nizu sledbenika
		flag = 1;
		for(int i=0;i<6;i++){
			if(Jobs[*p_id].succesors_id[i] == 0){
				flag = 0;
			}
		}
		if(flag == 1){
			printf("Greska, p_id vec ima 5 sledbenika! \n");
			fflush(stdout);
			return;
		}

		for(int i = 0; i<6; i++){
			if(Jobs[*p_id].succesors_id[i] == 0){
				pos_sl = i;
				break;
			}
		}

		int pos_pr;				// pozicija u nizu prethodnika
		flag = 1;
		for(int i=0;i<6;i++){
			if(Jobs[*s_id].predecesors_id[i] == 0){
						flag = 0;
			}
		}
		if(flag == 1){
			printf("Greska, p_id vec ima 5 sledbenika! \n");
			fflush(stdout);
			return;
		}
		for(int i = 0; i<6; i++){
			if(Jobs[*s_id].predecesors_id[i] == 0){
				pos_pr = i;
				break;
			}
		}

		if(rek_ciklicni_preth(p_id, s_id)){
			printf("Pojavljuje se ciklus!");
			fflush(stdout);
			return;
		}
		flag_rek = 0;

		int flag_za_vezup = 0;
		for(int i = 0; i<6; i++){	// proveri ponovo, sta ako dodajes samo naslednika?
			if(Jobs[*p_id].succesors_id[i] == *s_id){
				flag_za_vezup = 1;
			}
		}
		int flag_za_vezus = 0;
		for(int i = 0; i<6; i++){	// proveri ponovo, sta ako dodajes samo naslednika?
			if(Jobs[*s_id].predecesors_id[i] == *p_id){
				flag_za_vezus = 1;
			}
		}

		if(flag_za_vezup == 0){
			Jobs[*p_id].succesors_id[pos_sl] = *s_id;
		}
		if(flag_za_vezus == 0){
			Jobs[*s_id].predecesors_id[pos_pr] = *p_id;
		}


}

static void disconnect_jobs (int* p_id, int* s_id){

	printf("%d %d", *p_id, *s_id);

	if(((*p_id && *s_id) == 0) || (*p_id == *s_id)){ // da nisu 0;
		printf("Greska, niste validno uneli parametre! \n");
		return;
	}

	if(Jobs[*p_id].id != *p_id){
		printf("Ne postoji p_id! \n");
		return;
	}
	if(Jobs[*s_id].id != *s_id){
		printf("Greska, ne postoji s_id! \n");
		return;
	}

	int flag = 1;
	int pos_sl;
	for(int i = 0; i < 6; i++){
		if(Jobs[*p_id].succesors_id[i] == *s_id){
			flag = 0;
			pos_sl = i;
		}
	}
	if(flag == 1){
		printf("Ne postoji sledbenik!");
		return;
	}
	flag = 1;
	int pos_pr;
	for(int i = 0; i < 6; i++){
		if(Jobs[*s_id].predecesors_id[i] == *p_id){ // da li su povezani
			flag = 0;
			pos_pr = i;
		}
	}
	if(flag == 1){
		printf("Ne postoji prethodnik!");
		return;
	}

	Jobs[*p_id].succesors_id[pos_sl] = 0;
	Jobs[*s_id].predecesors_id[pos_pr] = 0;

	printf("Uspesan diskonekt!\n");
	fflush(stdout);

}

static void remove_job(int* job_id){

	if(*job_id == 0 || *job_id>count_id){
		printf("Niste validno uneli parametre!\n");
		fflush(stdout);
		return;
	}

	if(Jobs[*job_id].id != *job_id){
		printf("Posao ne postoji!\n");
		fflush(stdout);
		return;
	}

	for(int i = 0; i<6;i++){
		if(Jobs[*job_id].predecesors_id[i] != 0){
			disconnect_jobs(&Jobs[*job_id].predecesors_id[i], job_id);
		}
	}
	for(int i = 0; i<6;i++){
		if(Jobs[*job_id].succesors_id[i] != 0){
			printf("prvi: %d, drugi %d\n\n", *job_id, Jobs[*job_id].succesors_id[i]);
			disconnect_jobs(job_id, &Jobs[*job_id].succesors_id[i]);
			printf("prvi: %d, drugi %d", *job_id, Jobs[*job_id].succesors_id[i]);
			fflush(stdout);
		}
	}


	Jobs[*job_id].id = 0;

	printf("Uspesno diskonektovanje!\n");
	fflush(stdout);


}

static void list_jobs(){


	for(int i = 1; i<count_id; i++){

		if(Jobs[i].id == 0){
			continue;
		}

		printf("%d | %s | %s | ", Jobs[i].id, Jobs[i].title, Jobs[i].type);
		fflush(stdout);

		for(int j = 0; j<6; j++){
			if(Jobs[i].predecesors_id[j] == 0){
				printf("- ");
				continue;
			}

			printf("%d ", Jobs[i].predecesors_id[j]);
		}

		printf("| ");
		fflush(stdout);

		for(int k = 0; k<6; k++){
			if(Jobs[i].succesors_id[k] == 0){
				printf("- ");
				continue;
			}

			printf("%d ", Jobs[i].succesors_id[k]);
		}

		printf("|\n");
		fflush(stdout);
	}
}

static void xInterakcijaSaSistemom(void* pvParametri){

	for(;;){

		int choice;
		char name[100];
		char type[100];

		printf("Your options: \n 1: add_job <name> <type> \n2: connect_jobs <predecesor_id> <successor_id> \n"
				"3: disconnect_jobs <predecesor_id> <successor_id>\n4: remove_job <job_id> \n5: list_jobs\n");
		fflush(stdout);

		scanf("%d" , &choice);

		if(choice == 1){

			printf("Unesite ime i tip: \n");
			fflush(stdout);

			scanf("%s%s", name, type);

			int dodaj_posao = add_job(name, type);

			if(dodaj_posao){
				printf("Uspesno dodavanje!\n");
				fflush(stdout);
			}
		}else if(choice == 2){
			printf("Povezi dva posla preko id: ");
			fflush(stdout);

			int p_id, s_id;
			scanf("%d%d", &p_id,&s_id);

			connect_jobs(&p_id, &s_id);


		}else if(choice == 3){
			printf("Unesite dva id-a za diskonekt: \n");
			fflush(stdout);

			int p_id, s_id;
			scanf("%d%d", &p_id, &s_id);
			disconnect_jobs(&p_id, &s_id);

		}else if(choice == 4){
			printf("Unesite id za brisanje: \n");
			fflush(stdout);
			int job_id;

			scanf("%d", &job_id);
			remove_job(&job_id);

		}
		else if(choice == 5){
			list_jobs();
		}else{
			printf("Pogresna komanda!\n");
			fflush(stdout);
		}



	}

	vTaskDelete(0);
}


int domaci( void )
{

	xTaskCreate(xInterakcijaSaSistemom, "", configMINIMAL_STACK_SIZE, NULL, 1, NULL);

	vTaskStartScheduler();



	return 0;

}

