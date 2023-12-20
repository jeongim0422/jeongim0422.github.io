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


<br>
<br>

### c. 조합 시스템

- 아이템은 허브, 물병, 부산물, 기타로 종류가 나뉜다. 조합 UI는 최대 네 종류의 아이템을 필요로 하며 각 칸에는 특정 종류의 아이템만 넣을 수 있다. 아이템을 드래그해 시스템 슬롯에 넣으면 그에 적합한 레시피를 검사해 제작된 물약이 생성된다.

<br>

조합 슬롯에 적용된 PharmacySlot.cs는 드롭된 아이템이 슬롯이 허용하는 아이템 타입인지를 검사하고 허용 가능하다면 정보를 저장한다.

<br>
<br>

PharmacySlot.cs

<hr>

~~~
     
    public class PharmacySlot : MonoBehaviour, IDropHandler//, IPointerClickHandler, IBeginDragHandler, IDragHandler, IEndDragHandler, 
{
    // 허용할 아이템 타입 지정
    public Item.ItemType allowedType;

    public Item item; //획득 아이템
    public string itemName; // 아이템 이름
    public int itemCount; //획득 아이템 개수
    public Image itemImage; //아이템 이미지
    public GameObject BaseImage; // 배경 이미지

    [SerializeField]
    private GameObject itemCountImage;
    [SerializeField]
    private Text text_Count; //아이템 개수 UI로 띄우기 위해

    // 드래그 슬롯 정보 받아오기 위해 사용
    PharmacySlotManager pharmacySlotManager;

    private Slot previousDragSlot; // 슬롯의 아이템을 교체시 개수 다시 원상복귀시키기 위해 이전 아이템 저장

    // 슬롯 개수 원상복구시 메서드 호출 위해 사용
    public Inventory inventory;

    public ItemDatabase itemDatabase;

    public void OnDrop(PointerEventData eventData)
    {
        Slot dragSlot = eventData.pointerDrag.GetComponent<Slot>();

        // 드래그된 아이템이 허용하는 타입이며 한 개만 필요한 Bottle 타입이라면
        if (dragSlot.item != null && dragSlot.item.itemType == allowedType && allowedType == Item.ItemType.Bottle)
        {
            // 비어있는 상태였다면 조합 슬롯에 드래그한 슬롯 정보 추가하도록 함
            if (item == null)
            {
                item = dragSlot.item;
                itemName = item.itemName;
                itemImage.sprite = item.itemImage;
                itemCount = 1;

                BaseImage.SetActive(false);
                SetColor(1);
                itemDatabase.AddItem(dragSlot.item, -1);
                inventory.AddItemsInSlot();

                previousDragSlot = dragSlot; // 사본 저장

                dragSlot = null;
            }
            // 슬롯이 채워져 있으며, 드래그 슬롯과 현재 슬롯의 이름이 다르다면
            else if (item != null && dragSlot.item.itemName != item.itemName)
            {
                // 슬롯을 업데이트
                item = dragSlot.item;
                itemName = item.itemName;
                itemImage.sprite = item.itemImage;
                itemCount = 1;
                itemDatabase.AddItem(item, -1);
                // 이전에 선택했던 슬롯을 원상복구
                itemDatabase.AddItem(previousDragSlot.item, 1);

                previousDragSlot = dragSlot; // 사본 저장

                dragSlot = null;
            }
        }

        // Bottle이 아닌 개수가 표시되는 허용된 타입의 슬롯이라면
        else if (dragSlot.item != null && dragSlot.item.itemType == allowedType)
        {
            // 비어있는 상태였다면 바로 추가하도록 함
            if (item == null)
            {
                item = dragSlot.item;
                itemName = item.itemName;
                itemImage.sprite = item.itemImage;
                itemCount = 1;

                BaseImage.SetActive(false);
                SetColor(1);
                itemDatabase.AddItem(dragSlot.item, -1);
                inventory.AddItemsInSlot();
                // 상점 인벤토리 업데이트

                previousDragSlot = dragSlot; // 사본 저장

                dragSlot = null;
            }

            // 슬롯이 채워져 있으며, 드래그 슬롯과 현재 슬롯이 같다면(개수를 추가한다면)
            else if (item != null && dragSlot.item.itemName == item.itemName)
            {

                itemCount += 1;
                // 카운트 이미지 활성화
                itemCountImage.SetActive(true);
                text_Count.text = itemCount.ToString();
                itemDatabase.AddItem(dragSlot.item, -1);
                inventory.AddItemsInSlot();
                // 상점 인벤토리 업데이트

                previousDragSlot = dragSlot; // 사본 저장

                dragSlot = null;
            }

            // 슬롯이 채워져 있으며, 드래그 슬롯과 현재 슬롯의 이름이 다르다면
            else if (item != null && dragSlot.item.itemName != item.itemName)
            {
                // 슬롯을 업데이트
                item = dragSlot.item;
                itemName = item.itemName;
                itemImage.sprite = item.itemImage;
                itemCount = 1;
                itemDatabase.AddItem(item, -1);

                // 이전에 선택했던 슬롯을 원상복구
                itemDatabase.AddItem(previousDragSlot.item, 1);

                inventory.AddItemsInSlot();
                // 상점 인벤토리 업데이트

                previousDragSlot = dragSlot; // 사본 저장

                dragSlot = null;
            }
        }
    }

    private void SetColor(float _alpha)
    {
        Color color = itemImage.color;
        color.a = _alpha;
        itemImage.color = color;
    }
}

~~~

<br>

조합 UI에 적용된 PharmacyPotion.cs는 조합 제작된 레시피와 슬롯에 저장된 아이템을 검사해 포션을 생성해 슬롯에 전달한다.

<br>
<br>

PharmacyPotion.cs

<hr>

~~~
     
   public class PharmacyPotion : MonoBehaviour
{
    public PharmacyRecipe recipe;
    public ClickPotion clickPotion;

    Item potion;

    public PharmacySlot bottleSlot;
    public PharmacySlot herbSlot;
    public PharmacySlot lootSlot;
    public PharmacySlot ingredientSlot;

    public GameObject BaseImage; // 배경 이미지
    public Image potionImage;

    bool checkBottle;
    bool checkHerb;
    bool checkLoot;
    bool checkIngredient;

    public GameObject pharmacySlotGrid;
    PharmacySlot[] pharmacySlots;

    private void Start()
    {
        pharmacySlots = pharmacySlotGrid.GetComponentsInChildren<PharmacySlot>();
    }

    public void CraftPotion()
    {
        if (CheckRecipe()) // 레시피에 필요한 아이템들이 슬롯에 있는지 확인
        {
            // 레시피에 해당하는 포션을 생성하여 슬롯에 배치
            CreatePotion();
        }
    }

    private bool CheckRecipe()
    {
        checkBottle = false;
        checkHerb = false;
        checkLoot = false;
        checkIngredient = false;

        // 레시피 병과 병 슬롯 모두 존재하고
        if (recipe.bottle != null && bottleSlot.item != null)
        {
            // 아이템 이름도 같다면
            if (recipe.bottle.itemName == bottleSlot.itemName)
            {
                checkBottle = true;
            }
        }
        // 레시피 병과 병 슬롯의 두 개 다 존재하지 않는다면
        if (recipe.bottle == null && bottleSlot.item == null)
        {
            checkBottle = true;
        }

        if (recipe.herb != null && herbSlot.item != null)
        {
            if (recipe.bottle.itemName == herbSlot.itemName)
            {
                checkHerb = true;
            }
        }
        if (recipe.herb == null && herbSlot.item == null)
        {
            checkHerb = true;
        }

        if (recipe.loot != null && lootSlot.item != null)
        {
            if (recipe.loot.itemName == lootSlot.itemName)
            {
                checkLoot = true;
            }
        }
        if (recipe.loot == null && lootSlot.item == null)
        {
            checkLoot = true;
        }

        if (recipe.ingredient != null && ingredientSlot.item != null)
        {
            if (recipe.ingredient.itemName == ingredientSlot.itemName)
            {
                checkIngredient = true;
            }
        }
        if (recipe.ingredient == null && ingredientSlot.item == null)
        {
            checkIngredient = true;
        }

        return (checkBottle && checkHerb && checkLoot && checkIngredient);

    }

    // 포션을 생성하여 슬롯에 배치하는 메서드
    private void CreatePotion()
    {
        BaseImage.SetActive(false);

        potion = recipe.potion;

        potionImage.sprite = potion.itemImage;

        clickPotion.SetItem(potion);

        SetColor(potionImage, 1);

        ClearPharmacySlot();
    }

    public void ClearPotion()
    {
        BaseImage.SetActive(true);
        SetColor(potionImage, 0);
        potion = null;
        potionImage.sprite = null;
    }

    private void SetColor(Image image, float _alpha)
    {
        Color color = image.color;
        color.a = _alpha;
        image.color = color;
    }

    private void ClearPharmacySlot()
    {
        foreach (PharmacySlot pharmacySlot in pharmacySlots)
        {
            pharmacySlot.item = null;
            pharmacySlot.itemName = null;
            pharmacySlot.itemImage.sprite = null;
            pharmacySlot.itemCount = 0;

            SetColor(pharmacySlot.itemImage, 0);
            pharmacySlot.BaseImage.SetActive(true);
        }
    }
}

~~~


<br>
<br>

### d. 대화와 퀘스트 시스템

- NPC를 클릭하면 저장된 대화가 나타나 퀘스트를 얻게 된다.

<br>

빈 오브젝트에 부착된 DialogueManager.cs를 통해 대화 데이터와 초상화 정보를 저장한다.

<br>
<br>

DialogueManager.cs

<hr>

~~~
     
   public class DialogueManager : MonoBehaviour
{
    //여러 문장이 들어있으므로 string[]사용
    Dictionary<int, string[]> talkData;
    Dictionary<int, Sprite> portraitData;

    // 사용한 초상화 이미지를 배열에 할당해줌
    public Sprite[] portaitArr;

    void Awake()
    {
        talkData = new Dictionary<int, string[]>();
        portraitData = new Dictionary<int, Sprite>();
        GenerateData();
    }

    // 대화 데이터 추가
    void GenerateData()
    {
        // 초상화 정보를 추가해줌
        portraitData.Add(1000 + 0, portaitArr[0]);
        portraitData.Add(1000 + 1, portaitArr[1]);

        // 일반 대화
        talkData.Add(1000, new string[] { "안녕:0" });

        // 퀘스트용 대화, 퀘스트ID + NPCID
        talkData.Add(10 + 1000, new string[] { "어서 와.:0",
                                               "퀘스트를 줄게:1",
                                               "파란 보석을 하나 가져다 줘:0"});
        talkData.Add(11 + 1000, new string[] { "보석을 가져온거야?:0",
                                               "고마워:1"});
    }

    public string GetDialogue(int dialogueId, int talkIndex)
    {
        // 퀘스트에 맞는 대사가 없다면
        if (!talkData.ContainsKey(dialogueId))
        {
            // npc의 퀘스트 해당 퀘스트 관련 처음 대사가 없으면 기본 대사를 가져옴
            if (!talkData.ContainsKey(dialogueId - dialogueId % 10))
                return GetDialogue(dialogueId - dialogueId % 100, talkIndex);

            // npc의 퀘스트 해당 퀘스트 관련 처음 대사를 가져옴
            else
                return GetDialogue(dialogueId - dialogueId % 10, talkIndex);
        }

        // 대화가 끝이라면
        if (talkIndex == talkData[dialogueId].Length) return null;
        else return talkData[dialogueId][talkIndex];
    }

    public Sprite GetPortrait(int npcId, int portraitIndex)
    {
        return portraitData[npcId + portraitIndex];
    }
}

~~~

<br>

빈 오브젝트에 부착된 QuestManager.cs는 퀘스트 정보와 아이템을 저장하고 대화와 퀘스트 순서를 업데이트한다.

<br>
<br>

QuestManager.cs

<hr>

~~~
     
    public class QuestManager : MonoBehaviour
{
    public int questId;
    public int questIndex; //퀘스트 순서 구분
    public GameObject inventory;

    Dictionary<int, QuestData> questList;

    public Item[] questItems;

    Slot[] slots;

    void Awake()
    {
        questList = new Dictionary<int, QuestData>();
        questIndex = 0;
        slots = inventory.GetComponentsInChildren<Slot>();
        GenerateData();
    }

    // 퀘스트 정보 저장
    private void GenerateData()
    {
        questList.Add(10, new QuestData("포션 제조하기", new int[] { 1000 }, questItems[0]));
        questList.Add(20, new QuestData("퀘스트 올 클리어!", new int[] { 0 }, questItems[0]));
    }

    // NPCID를 받아 퀘스트 대사 코드 반환
    public int GetQuestDialogueIndex(int npcId)
    {
        return questId + questIndex;
    }

    public string CheckQuest(int npcId)
    {

        // 퀘스트 관련 대화하는 npc 수가 여러 명일 때
        if (questList[questId].npcId.Length > 1)
        {
            // 대화를 나누고 있는 npc가 퀘스트리스트의 현재 진행중인 퀘스트ID의 퀘스트 데이터에서
            // 퀘스트 관련 NPC 배열의 퀘스트 대화 순서가 같을 때
            if (npcId == questList[questId].npcId[questIndex])
            {
                if (CheckItem(questList[questId].item))
                {
                    // 퀘스트 대화 순서가 증가함
                    questIndex++;
                }
            }
        }
        else
        {
            if (questIndex + 1 == questList[questId].npcId.Length)
            {
                if (CheckItem(questList[questId].item))
                {
                    Debug.Log("증가");
                    // 퀘스트 대화 순서가 증가함
                    questIndex++;
                    //NextQuest();
                }

            }
        }
        if (questList[questId].npcId.Length == 1)
        {
            //questIndex = 0;
        }

        //ControlObject();

        // 현재 퀘스트 대화가 끝나고 다음 퀘스트 실행



        // 진행 중인 퀘스트 이름을 반환해 어떤 퀘스트 중인지 알 수 있도록 함
        return questList[questId].questName;
    }

    public string CheckQuest()
    {
        return questList[questId].questName;
    }

    void NextQuest()
    {
        questId += 10;
        questIndex = 0;
    }

    bool CheckItem(Item item)
    {

        for (int i = 0; i < slots.Length; i++)
        {
            if (slots[i].item != null)
            {
                if (slots[i].item.itemName == item.itemName)
                {
                    // 아이템 감소
                    return true;

                }
            }
        }
        return false;
    }
}

~~~


<br>
<br>

### e. 아이템 데이터베이스 생성

- 인벤토리가 여러 개이므로 아이템을 한번에 업데이트 할 수 있도록 데이터베이스를 생성한다.

<br>

빈 오브젝트에 부착된 ItemDatabase.cs는 딕셔너리를 통해 아이템 정보와 개수를 저장하고, 개수를 업데이트한다.

<br>
<br>

ItemDatabase.cs

<hr>

~~~
     
public class ItemDatabase : MonoBehaviour
{
    public Inventory inventory;
    public ShopInventory shopInventory;
    // 아이템과 개수를 저장하는 딕셔너리
    public Dictionary<Item, int> itemDictionary;

    public void Awake()
    {
        itemDictionary = new Dictionary<Item, int>();
    }

    public void AddItem(Item item, int count = 1)
    {
        // 아이템이 이미 딕셔너리에 존재한다면
        if (itemDictionary.ContainsKey(item))
        {
            // 개수만 더해줌
            itemDictionary[item] += count;

            // 아이템 개수가 -가 된다면 0으로 보정
            if (itemDictionary[item] <= 0)
            {
                itemDictionary[item] = 0;

            }
        }
        // 아이템이 없다면 새로 저장
        else if (!itemDictionary.ContainsKey(item))
        {
            itemDictionary.Add(item, count);
        }

        inventory.AddItemsInSlot();
        shopInventory.AddItemsInSlot(item, count);
    }
}

~~~
