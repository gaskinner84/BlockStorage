---

copyright:
  years: 2014, 2018
lastupdated: "2018-03-16"

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}

# 스냅샷

스냅샷은 {{site.data.keyword.blockstoragefull}}의 기능입니다. 스냅샷은 특정 시점의 볼륨 컨텐츠를 나타냅니다. 스냅샷을 사용하면 성능에 영향을 주지 않고 최소한의 공간만 사용하면서도 데이터를 보호할 수 있는 데이터 보호를 위한 첫 번째 방어벽입니다. 사용자가 볼륨에서 스냅샷 기능을 사용하여 중요한 데이터를 실수로 수정하거나 삭제한 경우, 데이터는 스냅샷 사본에서 쉽고 빠르게 복원할 수 있습니다.

{{site.data.keyword.blockstorageshort}}에서는 다음과 같은 두 가지 방법으로 스냅샷을 작성할 수 있습니다. 각 스토리지 볼륨에 대해 자동으로 스냅샷 사본을 작성 및 삭제하는 구성 가능한 스냅샷 스케줄을 통해 작성할 수 있습니다. 사용자의 요구사항에 따라 추가로 스냅샷 스케줄을 작성하거나 수동으로 사본을 삭제 및 스케줄을 관리할 수도 있습니다. 두 번째 방법은 수동 스냅샷을 작성하는 방법입니다.

스냅샷 사본은 특정 시점의 볼륨 상태를 캡처하는 {{site.data.keyword.blockstorageshort}} LUN의 읽기 전용 이미지입니다. 스냅샷 사본은 작성 시간 및 스토리지 영역 모두에서 매우 효율적입니다. {{site.data.keyword.blockstorageshort}} 스냅샷 사본 작성은 볼륨 크기 또는 스토리지에서 활동 레벨에 상관없이 몇 초밖에 걸리지 않으며 일반적으로 1초 미만입니다. 스냅샷 사본이 작성되면 데이터 오브젝트에 대한 변경사항은 스냅샷 사본이 존재하지 않는 것처럼 오브젝트의 현재 버전에 대한 업데이트로 반영됩니다. 그러면서도 데이터의 사본은 완벽하게 안전하게 유지됩니다. 

스냅샷 사본은 성능 오버헤드를 발생시키지 않기 때문에 {{site.data.keyword.blockstorageshort}} 볼륨별로 최대 50개의 스케줄된 스냅샷 및 50개의 수동 스냅샷을 저장할 수 있으며 이들 모두는 데이터의 읽기 전용 및 온라인 버전으로 액세스할 수 있습니다.


스냅샷을 사용하여 다음을 수행할 수 있습니다.

- 시스템 중단 없이도 특정 시점의 복구 지점 작성
- 볼륨을 이전의 특정 시점으로 되돌리기

스냅샷을 작성하려면 볼륨의 일부 스냅샷 영역을 구매해야 합니다. 스냅샷 영역은 초기 볼륨 주문 중 또는 **볼륨 세부사항** 페이지의 초기 프로비저닝 후에 **조치** 드롭 다운 단추를 누르고 **스냅샷 영역 추가**를 선택하여 추가할 수 있습니다. 스케줄된 스냅샷과 수동 스냅샷은 스냅샷 영역을 공유하기 때문에 영역을 적절하게 주문하십시오. 세부사항 및 지침은 [스냅샷 주문](ordering-snapshots.html) 문서를 참조하십시오.

## 스냅샷 우수 사례

스냅샷 디자인은 고객 환경에 따라 달라집니다. 다음 디자인 고려사항을 사용하여 스냅샷 사본을 플랜하고 구현할 수 있습니다. 
- 	각 볼륨 또는 LUN에서 스케줄을 통해 최대 50개의 스냅샷을 작성하고, 수동으로 최대 50개를 작성할 수 있습니다. 
- 	너무 많은 스냅샷을 작성하지는 마십시오. 시간별, 일별 또는 주별 스냅샷을 스케줄하여 스케줄된 스냅샷 빈도가 RTP와 RPO의 필요에 적합하고 적용되는 비즈니스 요구사항에 맞도록 하십시오. 
- 	스냅샷 AutoDelete는 스토리지 이용 확장 제어에 사용되어야 합니다. <br/>
    **참고**: AutoDelete 임계값은 95%로 고정됩니다.
    
스냅샷은 실제 오프사이트 DR 복제 또는 장기 보유 백업을 위한 대체가 아님에 항상 유의하십시오.
    
##  스냅샷은 어떻게 디스크 공간을 이용합니까?

스냅샷 사본은 전체 파일이 아니라 개별 블록을 유지하여 디스크 소비를 최소화합니다. 스냅샷 사본은 활성 파일 시스템의 파일이 변경 또는 삭제되는 경우에만 추가 영역을 사용하기 시작합니다. 이런 경우, 원본 파일 블록은 하나 이상의 스냅샷 사본으로 계속 유지됩니다.
활성 파일 시스템에서, 변경된 블록은 디스크의 다른 위치에 다시 기록되거나 활성 파일 블록으로 완전히 제거됩니다. 따라서, 수정된 활성 파일 시스템의 블록에서 사용되는 디스크 영역뿐만 아니라 원본 블록에서 사용되는 디스크 영역도 여전히 변경 전의 활성 파일 시스템 상태를 반영하도록 예약됩니다.

<table>
    <colgroup>
      <col style="width: 33.3%;"/>
      <col style="width: 33.3%;"/>
      <col style="width: 33.3%;"/>
    </colgroup>
    <tbody>
      <tr>
        <th colspan="3" style="border: 0.0px;text-align: center;">스냅샷 복사 전후 디스크 영역 사용량</th>
     </tr><tr>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle1.png" alt="스냅샷 복사 전"></td>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle3.png" alt="스냅샷 복사 후"></td>
        <td style="border: 0.0px;text-align: center;"><img src="/images/bfcircle2.png" alt="스냅샷 복사 후 변경"></td>
     </tr><tr>
        <td style="border: 0.0px;">임의 스냅샷 사본이 작성되기 전에는 활성 파일 시스템만 디스크 영역을 사용합니다.</td>
        <td style="border: 0.0px;">스냅샷 사본이 작성되면 활성 파일 시스템 및 스냅샷 사본이 동일한 디스크 블록을 지시합니다. 스냅샷 사본은 추가 디스크 영역을 사용하지 않습니다.</td>
        <td style="border: 0.0px;">활성 파일 시스템에서 <i>myfile.txt</i>가 삭제된 후에도 스냅샷 사본은 여전히 파일을 포함하고 해당 디스크 블록을 참조합니다. 그렇기 때문에 활성 파일 시스템 데이터를 삭제해도 디스크 영역이 항상 다시 사용 가능하지는 않습니다.</td>
      </tr>
    </tbody>
</table>

사용된 스냅샷 영역을 보려면 [스냅샷 관리](working-with-snapshots.html) 문서의 지시사항에 따라 수행하십시오.







