#include <stdio.h>
#include <kvm.h>
#include <limits.h>
#include <sys/param.h>
#include <sys/sysctl.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
struct structura_pt_proc{
        int nivel;
        struct kinfo_proc info_proc;
};
void DFS(int id_proc_curent, struct structura_pt_proc proc_dfs[],int nentries)
{
        int i;
        int nivel_tata;
        for(i=0;i<nentries;i++){
                if(proc_dfs[i].info_proc.p_pid==id_proc_curent){
                        int j;
                        nivel_tata=proc_dfs[i].nivel;
                        if(proc_dfs[i].nivel==0)
                                printf("%s", "->");
                        else{
                                for(j=0;j<proc_dfs[i].nivel;j++){
                                        printf("%s", "  |");
                                }
                        printf("%s", "_____");
                        }
			printf("%d %s %d %d\n",proc_dfs[i].info_proc.p_pid, proc_dfs[i].info_proc.p_comm,proc_dfs[i].info_proc.p_stat, proc_dfs[i].info_proc.p_pctcpu);
                }
        }
        for(i=0;i<nentries;i++){
                if(proc_dfs[i].info_proc.p_ppid==id_proc_curent){
                        proc_dfs[i].nivel=nivel_tata+1;
                        DFS(proc_dfs[i].info_proc.p_pid, proc_dfs, nentries);
                }
        }
}
int main(int argc, char** argv)
{
        int the_pid = strtol(argv[1], NULL, 10);
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kernel = kvm_openfiles(NULL, NULL, NULL, KVM_NO_FILES, errbuf);
        int nentries = 0;
struct kinfo_proc *kinfo=kvm_getprocs(kernel,KERN_PROC_ALL,0,sizeof(struct kinfo_proc), &nentries);
        int i;
        struct structura_pt_proc proc_dfs[nentries];
        for(i=0;i<nentries;i++){
                proc_dfs[i].info_proc=kinfo[i];
                proc_dfs[i].nivel=0;
        }
        DFS(the_pid, proc_dfs, nentries);
        return 0;
}
