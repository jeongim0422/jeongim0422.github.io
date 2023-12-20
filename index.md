<br>
<br>

# 프로젝트명 :  신(Sin) (개발자 : 권정임)

<br>
<br>
<br>

# **목차**         

### [1. 시스템 소개](#시스템-소개)
### [2. 게임 개발](#게임-개발)
- a. [타이밍 바를 통한 채집 시스템](#채집)
- b-1. [상점 구매 시스템](#구매)
- b-2. 상점 판매 시스템](#판매)
- c. [조합 시스템](#조합)
- d. [대화와 퀘스트 시스템](#대화)
- e. [아이템 데이터베이스 생성](#아이템)


<br>
<br>
<br>

# **[1. 시스템 소개]**

<br>

- 플레이어는 허브 채집과 몬스터 사냥을 통해 아이템을 얻는다. 해당 아이템을 상점에 판매하며 돈을 벌어 상점에서 아이템을 구매할 수 있다. NPC와의 대화를 통해 퀘스트를 수행할 수 있으며, 퀘스트를 수행하기 위해 획득한 아이템으로 포션을 제조한다.

- 

<br>
<br>
<br>

# **[2. 게임 개발]**

<br>

### a. 타이밍 바를 통한 채집 시스템

- 플레이어가 배치된 허브 오브젝트 가까이에서 X키를 누르면 채집 타이밍바가 허브 오브젝트 상단에 나타난다. 좌우로 이동하는 핸들에 맞춰 스페이스바를 누르면 핸들이 어떤 영역에 위치했는지에 따라 허브 프리팹을 생성한다.

<br>

허브 프리팹에는 Herb.cs 가 적용되어 타이밍바를 활성화하고 그 결과에 따라 프리팹을 인스턴스화한다.

<br>
Herb.cs
<hr>

~~~ 
    public class Herb : MonoBehaviour
{
    public GameObject herbPrefab; // 씬에 생성할 프리팹 할당
    Vector3 herbDropPos; //  허브 생성 위치 저장
    public int herbNum;
    int number;

    public TimingBar timingBar;
    public Slider timingSlider;

    Rigidbody h_Rigidbody;

    bool isCoroutineRunning = false;

    void Start()
    {
        herbDropPos = transform.position;
        h_Rigidbody = GetComponent<Rigidbody>();
    }

    public void TimingCall()
    {
        MoveUIAboveHerb();
        StartCoroutine(TimingAndCollect());
    }

    private void MoveUIAboveHerb()
    {
        timingSlider.gameObject.SetActive(true);

        timingBar.transform.position = Camera.main.WorldToScreenPoint(this.transform.position + new Vector3(1.2f, 2.5f, 0));
    }

    private IEnumerator TimingAndCollect()
    {
        if (!isCoroutineRunning)
        {
            isCoroutineRunning = true;

            timingSlider.value = 0;

            yield return StartCoroutine(timingBar.Timing());
            isCoroutineRunning = false;

            timingSlider.gameObject.SetActive(false);
        }

        if (timingBar.isGood)
        {
            number = 2;
            Collect(number);
        }
        else if (timingBar.isNice)
        {
            number = 1;
            Collect(number);
        }

        timingBar.resetTiming();
    }

    void Collect(int herbNum)
    {
        for (int num = 0; num < herbNum; num++)
        {
            Vector3 randomOffset = new Vector3(Random.Range(-0.5f, 0.5f), 0f, Random.Range(-0.5f, 0.5f)); // 랜덤 범위값 지정
            Instantiate(herbPrefab, herbDropPos + randomOffset, Quaternion.identity); // 드랍 아이템 생성
            h_Rigidbody.AddForce(Vector3.up * 0.1f, ForceMode.Impulse); // 생성 후 y축으로 힘을 가해 튀어오르듯 함
        }

        Destroy(gameObject, 2);
    }
}
~~~

<br>

타이밍바 UI에는 TimingBar.cs가 적용되어 핸들을 좌우로 이동시키고 슬라이더 값과 영역 크기를 비교해 결과값을 Herb.cs에 반환한다.
<br>
TimingBar.cs
<hr>

~~~
     
    public class TimingBar : MonoBehaviour
{
    public Slider timingSlider; // 타이밍바 슬라이더 할당

    public RectTransform goodTiming; // good 영역 이미지의 위치

    public int speed = 1000000; // 타이밍 속도 ♣ 추후 허브별 다르게 적용

    public float goodMinPos; // good 영역 시작 위치
    public float goodMaxPos; // good 영역 종료 위치

    public bool isNice; // 판정 결과 저장
    public bool isGood;
    public bool isGreat;

    private bool moveRight = true; // 초기에 우측으로 이동하도록 함


    void Awake()
    {
        timingSlider.value = 0; // 값 0으로 초기화
        goodMinPos = goodTiming.anchoredPosition.x; // good 영역 시작 position
        goodMaxPos = goodTiming.sizeDelta.x + goodMinPos; // good 영역 종료 position, 사이즈 x크기 + 시작위치

        resetTiming();
    }

    public IEnumerator Timing()
    {
        yield return null;

        while (!(Input.GetKeyDown(KeyCode.Space))) // 스페이스가 눌리지 않을 때까지
        {
            if (moveRight)
            {
                timingSlider.value += Time.deltaTime * speed; // 우측 이동

                if (timingSlider.value >= timingSlider.maxValue)
                {
                    timingSlider.value = timingSlider.maxValue;
                    moveRight = false; // 우측 이동이 완료되면 좌측으로 이동
                }
            }
            else if (!moveRight)
            {
                timingSlider.value -= Time.deltaTime * speed; // 좌측 이동

                if (timingSlider.value <= timingSlider.minValue)
                {
                    timingSlider.value = timingSlider.minValue;
                    moveRight = true; // 우측 이동이 완료되면 좌측으로 이동
                }
            }

            yield return null;
        }

        if (timingSlider.value >= goodMinPos && timingSlider.value <= goodMaxPos) // 스페이스 눌렀을 떄 value가 good 영역 사이라면
        {
            isGood = true;
        }
        else
        {
            isGood = false;
            isNice = true;
        }

        yield return null;
    }

    public void resetTiming()
    {
        timingSlider.value = 0;

        isNice = false;
        isGood = false;
        isGreat = false;
    }
}

~~~
