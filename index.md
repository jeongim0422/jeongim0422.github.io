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
- b-2. [상점 판매 시스템](#판매)
- c. [조합 시스템](#조합)
- d. [대화와 퀘스트 시스템](#대화)
- e. [아이템 데이터베이스 생성](#아이템)


<br>
<br>
<br>

# **[1. 시스템 소개]**

<br>

- 플레이어는 허브 채집과 몬스터 사냥을 통해 아이템을 얻는다. 해당 아이템을 상점에 판매하며 돈을 벌어 상점에서 아이템을 구매할 수 있다. NPC와의 대화를 통해 퀘스트를 수행할 수 있으며, 퀘스트를 수행하기 위해 획득한 아이템으로 포션을 제조한다.

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


<br>
<br>

### b-1. 상점 구매 시스템

- 상점은 판매 아이템 프리팹을 미리 배열에 할당한다. 이후 Start() 메서드가 실행되면 배열 정보를 받아 슬롯 프리팹을 생성해 스크롤뷰에 넣어준다. 아이템을 구매하면 플레이어의 금액이 감소되며 인벤토리에 해당 아이템이 추가된다.

<br>

Shop.cs는 상점 UI의 상위 오브젝트에 할당되며 저장된 배열 정보에 따라 슬롯을 생성한다. 구매 버튼을 클릭한다면 구매 가능 여부에 따라 아이템을 인벤토리에 추가한다. 또한 슬롯을 클릭 시 해당 슬롯에 선택 이미지가 나타나도록 한다.

<br>
<br>

Shop.cs

<hr>

~~~
     
    public class Shop : MonoBehaviour
{
    public GameObject shopItemSlotPrefab; // 아이템 슬롯 UI 프리팹, 미리 할당되어 있음
    public Transform content;   // 아이템 슬롯을 담은 상위 오브젝트
    public ScrollRect scrollRect;

    public ItemDatabase itemDatabase;

    public Inventory inventory;
    public ShopInventory shopInventory;
    public Inventory pharmacyInventory;
    public PlayerMoney playerMoney;

    bool canBuy;

    RectTransform contentRectTransform;
    RectTransform shopSlotRectTransform;

    GameObject itemSlotObj;
    ShopSlot shopSlot;

    public Item[] shopItems; // 상점에 있는 모든 아이템 스크립터블 오브젝트를 배열에 할당해줌
    private ShopSlot[] shopSlotsArray; // 상점의 슬롯들을 담을 배열

    public Image selectedImage;
    private int selectedSlotIndex;  // 선택된 슬롯의 인덱스

    void Start()
    {
        canBuy = false;
        InitializeShopUI();
        shopSlotsArray = content.GetComponentsInChildren<ShopSlot>();
        selectedSlotIndex = 0;
    }

    void InitializeShopUI()
    {
        contentRectTransform = content.GetComponent<RectTransform>();
        float totalHeight = 0f; // Content의 초기 높이를 나타내는 변수

        for (int index = 0; index < shopItems.Length; index++)
        {
            itemSlotObj = Instantiate(shopItemSlotPrefab, content); // 아이템 개수만큼 슬롯 생성함
            shopSlot = itemSlotObj.GetComponent<ShopSlot>(); // 생성된 각 슬롯의 스크립트를 가져옴

            shopSlot.SetItem(shopItems[index], index); // 아이템 정보 설정

            shopSlotRectTransform = shopSlot.GetComponent<RectTransform>();
            totalHeight += shopSlotRectTransform.sizeDelta.y;

            // 슬롯이 추가될 때마다 Content의 높이를 slot의 높이만큼 증가시킴 
            //contentRectTransform.sizeDelta = new Vector2(contentRectTransform.sizeDelta.x, contentRectTransform.sizeDelta.y + shopSlotRectTransform.sizeDelta.y);
            contentRectTransform.sizeDelta = new Vector2(contentRectTransform.sizeDelta.x, contentRectTransform.sizeDelta.y + totalHeight);
        }
    }

    public void SelectSlot(int index)
    {

        // 슬롯 선택 시 호출되어 선택 이미지를 슬롯의 아이템 이미지로 이동시키고 투명도 조절해 보이도록 함
        selectedImage.transform.position = shopSlotsArray[index].Image_Item.transform.position;
        // 선택 이미지가 슬롯에 가려지지 않고, 스크롤뷰를 움직여도 슬롯 따라 움직이도록 함
        selectedImage.transform.SetParent(shopSlotsArray[index].transform);
        SelectSetColor(1);

        selectedSlotIndex = index;
    }

    private void SelectSetColor(float _alpha) // 슬롯 선택 시 선택 이미지 나타나도록 함
    {
        Color color = selectedImage.color;
        color.a = _alpha;
        selectedImage.color = color;
    }

    public void ClickBuyBtn()
    {
        canBuy = playerMoney.BuyItem(shopSlotsArray[selectedSlotIndex].item);

        if (canBuy)
        {
            itemDatabase.AddItem(shopSlotsArray[selectedSlotIndex].item);
            //pharmacyInventory.AcquireItem(shopSlotsArray[selectedSlotIndex].item);
        }
    }
}

~~~
