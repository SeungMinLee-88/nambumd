3-4 hpc
HPC(High Performance Computing)란

HPC는 여러 대의 서버(노드)를 묶어 하나의 큰 계산 문제를 병렬로 분산 처리하는 컴퓨팅 방식이다. 목표는 “유연한 서비스 운영”이 아니라 최대 처리량·최단 시간으로 대규모 연산을 수행하는 것이다.

1) 구성 요소
클러스터 구성

Compute Node: 실제 계산을 수행 (CPU/GPU, 메모리)

Login/Head Node: 사용자가 접속해 작업 제출, 모듈 로드, 빌드 등

Scheduler/Resource Manager: 자원 할당 및 큐 관리 (예: SLURM, PBS)

고속 네트워크: 노드 간 통신 (예: InfiniBand / RoCE / 고성능 Ethernet)

공유 스토리지:

Home/Project 용: NFS, CephFS 등

고성능 I/O: 병렬 파일시스템(Lustre, Spectrum Scale/GPFS 등)

소프트웨어 스택(전형)

OS: Linux

컴파일러/라이브러리: GCC/Intel/LLVM, OpenMPI/MPICH, MKL/OpenBLAS

워크로드: MPI/OpenMP, CUDA, PyTorch/TensorFlow, CFD/FEA/분자동역학 등

환경 관리: Lmod(Environment Modules), Spack/Conda, 컨테이너(Singularity/Apptainer)

2) 병렬 처리 모델(핵심)
(1) 공유 메모리 병렬: OpenMP

한 노드 내부에서 여러 CPU 코어가 같은 메모리 공간을 공유하며 병렬 실행

(2) 분산 메모리 병렬: MPI

여러 노드가 각자 메모리를 가지고 실행

메시지로 데이터를 주고받음(노드 간 통신 성능이 중요)

(3) 하이브리드: MPI + OpenMP

노드 간은 MPI, 노드 내부는 OpenMP로 분업

(4) GPU 병렬: CUDA + 분산 학습(NCCL 등)

노드 내/노드 간 GPU 통신을 최적화해 대규모 학습 수행

3) HPC의 실행 방식(배치/큐)

HPC는 보통 “즉시 실행”이 아니라 다음 흐름이다.

로그인 노드에서 소스/스크립트 준비

배치 스크립트(job script) 작성

스케줄러에 제출(sbatch 등)

큐에서 대기 → 자원 확보 시 실행

로그/결과 파일 수집

구현 예시 1: MPI “Hello” (C) + SLURM 실행
1) MPI 코드 (hello_mpi.c)
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);

    int world_size, world_rank;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    printf("Hello from rank %d of %d\n", world_rank, world_size);

    MPI_Finalize();
    return 0;
}

2) 컴파일 (로그인 노드)
module load openmpi   # 환경에 따라 이름이 다를 수 있음
mpicc -O2 -o hello_mpi hello_mpi.c

3) SLURM 배치 스크립트 (job_mpi.sh)
#!/bin/bash
#SBATCH --job-name=mpi_hello
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4
#SBATCH --time=00:02:00
#SBATCH --output=out.%j.txt
#SBATCH --error=err.%j.txt

module load openmpi
srun ./hello_mpi

4) 제출/확인
sbatch job_mpi.sh
squeue -u $USER
cat out.<jobid>.txt


의미

--nodes=2: 2대 노드 사용

--ntasks-per-node=4: 노드당 MPI 프로세스 4개(총 8 rank)

srun: SLURM이 자원 배치와 실행을 통합 관리

구현 예시 2: OpenMP 병렬 (C) + 단일 노드 다중 코어
1) OpenMP 코드 (sum_omp.c)
#include <stdio.h>
#include <omp.h>

int main() {
    const long N = 100000000;
    double sum = 0.0;

    #pragma omp parallel for reduction(+:sum)
    for (long i = 1; i <= N; i++) {
        sum += 1.0 / (double)i;
    }

    printf("sum = %.10f\n", sum);
    return 0;
}

2) 컴파일/실행
gcc -O2 -fopenmp -o sum_omp sum_omp.c
export OMP_NUM_THREADS=16
./sum_omp

3) SLURM 배치 예 (job_omp.sh)
#!/bin/bash
#SBATCH --job-name=omp_sum
#SBATCH --nodes=1
#SBATCH --cpus-per-task=16
#SBATCH --time=00:02:00
#SBATCH --output=out.%j.txt

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
./sum_omp

구현 예시 3: PyTorch 분산 학습(DDP) + SLURM (GPU)

아래는 “노드 1대, GPU 여러 장” 기준의 가장 단순한 DDP 예시다. (멀티노드는 설정이 조금 더 추가됨)

1) PyTorch DDP 예제 (train_ddp.py)
import os
import torch
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

def main():
    dist.init_process_group(backend="nccl")
    local_rank = int(os.environ["LOCAL_RANK"])
    torch.cuda.set_device(local_rank)

    model = torch.nn.Linear(1024, 1024).cuda()
    ddp = DDP(model, device_ids=[local_rank])

    optim = torch.optim.Adam(ddp.parameters(), lr=1e-3)
    x = torch.randn(64, 1024, device="cuda")
    y = torch.randn(64, 1024, device="cuda")

    for step in range(100):
        optim.zero_grad(set_to_none=True)
        loss = torch.nn.functional.mse_loss(ddp(x), y)
        loss.backward()
        optim.step()
        if step % 20 == 0 and dist.get_rank() == 0:
            print("step", step, "loss", float(loss))

    dist.destroy_process_group()

if __name__ == "__main__":
    main()

2) SLURM 스크립트 (job_torch_ddp.sh)
#!/bin/bash
#SBATCH --job-name=torch_ddp
#SBATCH --nodes=1
#SBATCH --gres=gpu:4
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=8
#SBATCH --time=00:10:00
#SBATCH --output=out.%j.txt

module load cuda  # 환경에 따라 다름
# 가상환경/conda 활성화가 필요하면 여기서 수행

srun --ntasks=$SLURM_NTASKS \
     torchrun --standalone --nproc_per_node=4 train_ddp.py


포인트

--gres=gpu:4: GPU 4장 요청

--ntasks-per-node=4: 프로세스 4개(보통 GPU당 1 프로세스)

backend="nccl": GPU 분산 통신에 최적

4) HPC 구현에서 자주 부딪히는 이슈(실무 포인트)

NUMA/코어 바인딩: 성능 변동의 주원인

CPU affinity, --cpu-bind(SLURM) 등을 사용

네트워크 병목: 분산 학습·MPI에서 치명적

인터커넥트(IB/RoCE)와 튜닝 필요

스토리지 I/O: 데이터 로딩이 학습을 막는 경우 다수

로컬 SSD 캐시, 병렬 파일시스템, 데이터 샤딩

재현성/환경 관리: 모듈/Spack/컨테이너로 고정

스케줄링 정책: 공정성(페어쉐어), 프리엠션, 파티션 설계

5) “HPC 서비스” 관점에서 흔한 제공 항목

파티션(CPU/GPU), QoS, 계정/프로젝트 할당량

SLURM 기반 배치/인터랙티브(예: salloc) 제공

공용 스토리지 + 고속 병렬 스토리지

모듈/컨테이너 기반 SW 스택

모니터링/회계(사용량 집계), 정책(페어쉐어)

원하면 위 예시를 멀티노드 PyTorch DDP(2노드 이상) 형태로 확장한 SLURM 스크립트(마스터 주소/포트, rendezvous 설정 포함)까지 바로 제공할 수 있다.