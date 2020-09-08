# Logical Volume Manager (LVM)
- Logical volume management는 디스크나 대용량 스토리지 장치를 유연하고 확장이 가능하게 다룰 수 있는 기술이며 이를 커널에 구현한 기능을 바로 LVM(Logical Volume Manager) 라고 부른다. 
- LVM 은 이름처럼 파티션대신 볼륨이라는 단위로 저장 장치를 다룰 수 있으며, 물리 디스크를 볼륨 그룹으로 묶고 이것을 논리 볼륨으로 분할하여 관리한다. 
- 스토리지의 확장이나 변경시 서비스의 변경을 할 수 있으며 특정 영역의 사용량이 많아져서 저장 공간이 부족할 경우에 유연하게 대응할 수 있다. 

## LVM의 5가지 용어
- PV(Physical Volume)
- PE(Physical Extent)
- VG(Volume Group)
- LV(Logical Volume)
- LE(Logical Extent)

#### PV(Physical Volume)
- LVM에서 블록 장치 (블록 단위로 접근하는 스토리지, ex.하드디스크 ) 를 사용하려면 PV를 초기화 해야한다.
- LVM 으로 쓰기 위해서 PV로 초기화 하게 된다. PV는 일정 크기의 PE 들로 구성됨

#### PE(Physical Extent)
- PV를 구성하는 일정 크기의 블록단위로 LVM2에서는 4MB가 기본 값
- PE는 LV의 LE로 1:1로 대응 된다. 항상 PE 와 LE 의 크기는 동일함.

#### VG(Volume Group)
- PV 들의 집합이며 LV를 할당할 수 있는 공간이다.
- PV들로 초기화된 장치들은 VG로 관리된다.
- VG 안에서 사용자 임의로 공간을 쪼개 LV로 만드는것이 가능하다.

#### LV(Logical Volume)
- 사용자가 최종적으로 다루게되는 논리적인 스토리지 이다.
- 생성된 LV는 파일 시스템 혹은 애플리케이션 으로 사용된다.
- LV 는 3가지 유형이 있음

1. 선형 LV
    - 하나의 LV로 PV를 모으는 방법
    - 예를 들면 60GB 의 PV 2개로 120GB LV 를 만드는 방식
    - 사용자 입장에선 120GB 의 단일 장치만 있는 셈이다.
2. 스트라이프 LV
    - LV에 데이터 기록시, 파일 시스템이 PV에 데이터를 기록하게 되는데, 스트라이프된 LV를 생성해서 데이터가 PV에 기록되는 방식을 바꿀수 있음
    - 대량의 순차적 R/W 의 경우 I/O 효율을 높힐수 있는 방법
    - Round-Robin 방식으로 미리 지정된 PV 들에 데이터를 분산 기록해서 성능을 높히고, R/W 를 병렬로 실행할 수 있다.
3. 미러된 LV
    - 이름 그대로 블록 장치에 젖아된 데이터의 복사본을 다른 블록장치에 저장하는 방식.
    - 동일한 데이터를 저장함으로 써 장애 발생시 데이터를 보호할 수 있다.

#### LE(Logical Extent)
- LV 를 구성하는 기본 단위이고, 4MB이다.