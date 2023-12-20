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

<br>

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

<br>

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

<br>

상점 슬롯 프리팹에 적용되는 ShopSlot.cs는 슬롯에 아이템 정보를 넣어주고 슬롯이 클릭되면 그 슬롯의 인덱스를 Shop.cs에 반환해 적절한 슬롯에 선택 이미지가 나타날 수 있도록 한다.

<br>
<br>

ShopSlot.cs

<hr>

~~~
     
    public class ShopSlot : MonoBehaviour, IPointerClickHandler
{
    public Item item; //획득 아이템
    Shop shop;

    [SerializeField]
    private Text text_Item_Name; //아이템 이름 할당
    [SerializeField]
    private Text text_Item_Desc; //아이템 설명 할당
    public Image Image_Item; //아이템 이미지 할당
    public Text text_Item_BuyPrice;


    [SerializeField]
    private GameObject Prefab_Selected; // 선택 이미지 프리팹
    Image image_Selected; // 선택 이미지

    public int slotIndex = 0;

    void Awake()
    {
        shop = GetComponentInParent<Shop>();
    }

    public void SetItem(Item newItem, int index)
    {
        item = newItem;

        // UI 요소에 아이템 정보 설정
        text_Item_Name.text = item.itemName;
        text_Item_Desc.text = item.itemDesc;
        text_Item_BuyPrice.text = item.buyPrice.ToString();
        Image_Item.sprite = item.itemImage;

        slotIndex = index; // 슬롯이 본인의 인덱스 순서 갖도록 함
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        shop.SelectSlot(slotIndex);
    }
}

~~~

<br>

플레이어의 금액을 관리하는 PlayerMoney.cs는 구매와 판매에 따른 소지 금액을 UI에 업데이트하며 아이템 구매 시 구매 가능 여부를 Shop.cs에 반환한다.

<br>
<br>

PlayerMoney.cs

<hr>

~~~
     
    public class PlayerMoney : MonoBehaviour
{
    public int money;

    public Text text_Money;

    void Start()
    {
        money = 100;
    }

    private void Update()
    {
        text_Money.text = money.ToString();
    }

    public bool BuyItem(Item item)
    {
        if (money - item.buyPrice >= 0)
        {
            money -= item.buyPrice;

            //Debug.Log("구매 후 보유 금액 : " + money);

            return true;
        }

        else//else if (money - item.sellPrice < 0)
        {
            return false;
        }
    }

    public void SellItem(Item item)
    {
        money += item.sellPrice;
        //Debug.Log("판매 후 보유 금액 : " + money);
    }
}
~~~


<br>
<br>

### b-2. 상점 판매 시스템

- 상점 UI를 활성화하면 상점 인벤토리 역시 활성화된다. 인벤토리의 아이템을 클릭한 후 판매 버튼을 누르면 인벤토리의 아이템이 하나 감소되며 해당 금액이 소지 금액에 추가된다.

<br>

상점 인벤토리에 부착된 ShopInventory.cs는 인벤토리의 아이템에 따라 슬롯 프리팹을 생성하거나 제거하며 업데이트한다. 아이템 선택을 위해 슬롯이 제거된다면 슬롯을 인덱스를 재부여하고 금액을 업데이트한다.

<br>
<br>

ShopInventory.cs

<hr>

~~~
     
    public class ShopInventory : MonoBehaviour
{
    public GameObject shopItemSlotPrefab; // 아이템 슬롯 UI 프리팹, 미리 할당되어 있음
    public Transform content;   // 아이템 슬롯을 담은 상위 오브젝트
    public ScrollRect scrollRect;

    public PlayerMoney playerMoney;

    public ItemDatabase itemDatabase;
    public Inventory pharmacyInventory;

    RectTransform contentRectTransform;
    RectTransform shopInventorySlotRectTransform;

    GameObject itemSlotObj;
    ShopInventorySlot shopInventorySlot;

    [SerializeField]
    private GameObject inventoryContent;
    [SerializeField]
    private GameObject pharmacyInventoryContent;

    public Slot[] inventorySlots;
    public Slot[] pharmacyInventorySlots;

    //public Item[] inventoryItems; // 인벤토리의 아이템들을 담아줄 배열
    public List<ShopInventorySlot> shopInventorySlots = new List<ShopInventorySlot>(); // 인벤토리의 아이템들을 담아줄 리스트


    public Image selectedImage;
    private Image previousSelectedImage; // selectedImager가 null이 되는 것을 방지하기 위해 복사본 생성
    private int index;  // 슬롯의 인덱스
    int selectedSlotIndex; // 선택된 슬롯의 인덱스

    void Start()
    {
        inventorySlots = inventoryContent.GetComponentsInChildren<Slot>();
        pharmacyInventorySlots = pharmacyInventoryContent.GetComponentsInChildren<Slot>();
        contentRectTransform = content.GetComponent<RectTransform>();

        index = -1;
    }

    // 아이템 데이터베이스에 따라 슬롯을 업데이트
    public void AddItemsInSlot()
    {
        foreach (var item in itemDatabase.itemDictionary)
        {
            Item itemData = item.Key;
            int itemDataCount = item.Value;

            for (int i = 0; i < shopInventorySlots.Count; i++)
            {
                if (shopInventorySlots[i].item != null)
                {
                    if (shopInventorySlots[i].item.itemName == itemData.itemName)
                    {
                        shopInventorySlots[i].SetSlotCount(itemDataCount); //슬롯 아이템 개수 업데이트
                    }
                }
            }
        }
    }

    public void AddItemsInSlot(Item aquireItem, int count)
    {
        bool foundItem = false;

        foreach (var item in itemDatabase.itemDictionary)
        {
            Item itemData = item.Key;
            int itemDataCount = item.Value;

            // 획득한 아이템에 대해 개수 업데이트 위해 조건문 작성
            if (itemData == aquireItem)
            {
                for (int i = 0; i < shopInventorySlots.Count; i++)
                {
                    if (shopInventorySlots[i].item != null)
                    {
                        if (shopInventorySlots[i].item.itemName == itemData.itemName)
                        {
                            // 상점인벤에 아이템이 없을 때만 슬롯 생성하기 위해 foundItem = true 로 설정
                            foundItem = true;
                            shopInventorySlots[i].SetSlotCount(itemDataCount); //슬롯 아이템 개수 업데이트

                        }
                    }
                }
            }
        }
        // 획득 아이템이 슬롯에 없다면
        if (foundItem == false)
        {
            itemSlotObj = Instantiate(shopItemSlotPrefab, content); // 슬롯을 추가 생성
            shopInventorySlot = itemSlotObj.GetComponent<ShopInventorySlot>(); // 생성된 슬롯의 스크립트를 가져옴

            // 슬롯을 담은 리스트에 새로 생성한 리스트 추가
            shopInventorySlots.Add(shopInventorySlot);
            index = shopInventorySlots.Count - 1;

            for (int i = 0; i < shopInventorySlots.Count; i++)
            {
                if (shopInventorySlots[i].item == null)
                {
                    shopInventorySlots[i].AddItem(index, aquireItem, count);
                    break;
                }
            }
        }
    }

    public void RefreshSlot(int indexToRemove)
    {
        // 삭제할 슬롯의 인덱스가 0 이상이고 슬롯 리스트 개수보다 작을 때
        if (indexToRemove >= 0 && indexToRemove < shopInventorySlots.Count)
        {
            // 리스트에서 해당 인덱스의 슬롯 삭제
            shopInventorySlots.RemoveAt(indexToRemove);
        }

        // 리스트 순서대로 인덱스를 재부여
        for (int i = indexToRemove; i < shopInventorySlots.Count; i++)
        {
            shopInventorySlots[i].slotIndex = i;
        }


    }

    // 슬롯이 파괴되면 selectedImage가 null이 되는 것을 방지하기 위해 호출
    public void ClearSelectedImageReference(ShopInventorySlot destroySlot)
    {
        previousSelectedImage = selectedImage != null ? Instantiate(selectedImage) : null; // 이전 이미지의 복사본 생성

        destroySlot.gameObject.SetActive(false);
        Destroy(destroySlot.gameObject);
        //AddItemsInSlot();


        if (previousSelectedImage != null)
        {
            selectedImage = previousSelectedImage;
        }
    }

    public void SelectSlot(int index)
    {

        // 슬롯 선택 시 호출되어 선택 이미지를 슬롯의 아이템 이미지로 이동시키고 투명도 조절해 보이도록 함
        selectedImage.transform.position = shopInventorySlots[index].Item_Image.transform.position;
        // 선택 이미지가 슬롯에 가려지지 않고, 스크롤뷰를 움직여도 슬롯 따라 움직이도록 함
        selectedImage.transform.SetParent(shopInventorySlots[index].transform);
        SelectSetColor(1);

        selectedSlotIndex = index;
    }

    public void SelectSetColor(float _alpha) // 슬롯 선택 시 선택 이미지 나타나도록 함
    {
        Color color = selectedImage.color;
        color.a = _alpha;
        selectedImage.color = color;
    }

    public void ClickSellBtn()
    {
        if (selectedSlotIndex >= 0 && selectedSlotIndex < shopInventorySlots.Count)
        {
            // 인벤토리 아이템 감소 로직
            playerMoney.SellItem(shopInventorySlots[selectedSlotIndex].item); // 아이템 판매 처리
            itemDatabase.AddItem(shopInventorySlots[selectedSlotIndex].item, -1);

        }
    }
}

~~~
