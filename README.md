# Unity를 이용한 간단한 탄알 피하기 게임

+ 사용언어 : C#
+ 사용 소프트 웨어 : Unity
+ 제작기한 : 2022.11 ~ 2020.11 ( 이틀 소요 )
+ 제작인원 : 1명
--------------------------------------------------

+ 게임 화면

![탄알피하기 게임](https://user-images.githubusercontent.com/93432326/208557882-28a9c80c-16f9-4dc8-a6be-cd7a6068b05a.PNG)

+ **코드**

  + Bullet code
  
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bullet : MonoBehaviour
{

    public float speed = 8f; //탄알 이동속도
    private Rigidbody bulletRigidbody; // 이동에 사용할 리지드바디 컴포넌트
    void Start()
    {
        //게임 오브젝트에서 Rigidbody 컴포넌트를 찾아 BulletRigidbody에 할당
        bulletRigidbody = GetComponent<Rigidbody>();
        //리지드바디의 속도 = 앞쪽 방향 * 이동속력
        bulletRigidbody.velocity = transform.forward * speed;

        //3초 뒤에 자신의 게임 오브젝트 파괴
        Destroy(gameObject, 3f);
    }


   //트리거 출동 시 자동으로 실행되는 메소드
    private void OnTriggerEnter(Collider other)
    {
        //충돌한 상대방 게임 오브젝트가 Player 태그를 가진 경우
        if (other.tag == "Player")
        {
            // 상대방 게임 오브젝트에서 PlayerController 컴포넌트 가져오기
            PlayerController playerController = other.GetComponent<PlayerController>();

            //상대방으로부터 PlayerController 컴포넌트를 가져오는데 성공했다면
            if(playerController != null)
            {
                //상대방 PlayerController 컴포넌트의 Die() 메서드 실행
                playerController.Die();
            }
        }
    }

}

```

  + Bullet Spawner 코드 
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BulletSpawner : MonoBehaviour
{
    public GameObject bulletPrefab; // 생성할 탄알의 원본 프리팹
    public float spawnRateMin = 0.5f; // 최소 생성 주기
    public float spawnRateMax = 3f; // 최대 생성 주기

    private Transform target; //발사할 대상
    private float spawnRate; //생성 주기
    private float timeAfterSpawn; // 최근 생성 시점에서 지난 시간
    void Start()
    {
        //최근 생성 이후의 누적 시간을 0으로 초기화
        timeAfterSpawn = 0f;
        //탄알 생성 간격을 spawnRateMin과 spawnRateMax 사이에서 랜덤 지정
        spawnRate = Random.Range(spawnRateMin, spawnRateMax);
        //PlayerController 컴포넌트를 가진 게임 오브젝트를 찾아 조준대상으로 설정
        target = FindObjectOfType<PlayerController>().transform;
    }

    
    void Update()
    {
        //timeAfterSpawn 갱신
        timeAfterSpawn += Time.deltaTime;

        //최근 생성 시점에서부터 누적된 시간이 생성주기보다 크거나 같다면
        if (timeAfterSpawn >= spawnRate)
        {
            //누적된 시간 리셋
            timeAfterSpawn = 0f;

            //bulletPrefab의 복제본을 transform.position위치와 transform.rotation 회전으로 생성
            GameObject bullet = Instantiate(bulletPrefab, transform.position, transform.rotation);

            //생성된 bullet 게임 오브젝트의 정면 방향이 target을 향하도록 회전
            bullet.transform.LookAt(target);

            //다음번에 생성 간격을 spawnRateMin, spawnRateMax 사이에서 랜덤 지정
            spawnRate = Random.Range(spawnRateMin, spawnRateMax);
        }
    }
}

```

  + Player 코드
  
``` c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private Rigidbody playerRigidbody; // 이동에 사용할 리지드 바디 컴포넌트
    public float speed = 8f; // 이동속력

    void Start()
    {
        playerRigidbody = GetComponent<Rigidbody>();
    }


    void Update()
    {
        //수평축 , 수직축의 입력 값을 감지하여 저장
        float xInput = Input.GetAxis("Horizontal");
        float zInput = Input.GetAxis("Vertical");

        //실제 이동속도를 입력값과 이동속력을 사용해 결정
        float xSpeed = xInput * speed;
        float zSpeed = zInput * speed;

        //Vector 3 속도를 (xSpeed, 0f, zSpeed) 로 생성
        Vector3 newVelocity = new Vector3 (xSpeed, 0f, zSpeed);

        // 리지드바디의 속도에 newVelocity 할당
        playerRigidbody.velocity = newVelocity;
    }

    public void Die()
    {
        //게임 오브젝트 비활성화
        gameObject.SetActive(false);
    }
}


```

------------------------------------------------------------------------
  + 게임 실행 영상
https://user-images.githubusercontent.com/93432326/208560153-9cd43622-a8ce-4df5-be97-e7bbd70411f2.mp4

  + 프로젝트 설명 
    + Unity를 이용한 기본적인 탄알 피하기 게임
    + Bullet 스크립트를 이용해 탄알의 기본적인 속도, 컴포넌트, 생성 이후 삭제 시간을 설정
    + Bullet Spawenr 스크립트에서는 탄알의 생성간격, 발사할 대상(Player를 추적), Bullet 스크립트를 사용해 만든 프리팹 사용
    + Bullet Spawner 스크립트에서 플레이어의 움직임에 따라 플레이어의 위치에 따라 탄알을 발사
    + Player Controller 스크립트에서는 플레이어의 속도, 컴포넌트 설정
    + Player의 속도를 x축과 z축의 입력값과 이동속력을 이용해 결정 후 탄알에 충돌 후 오브젝트를 SetActive(false)를 통해 사망 구현

