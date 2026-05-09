---
title: Software Design Competition
published: 2026-04-03
description: 'notes of software design competition'
image: ''
tags: [C#, ASP.NET, Winform, Android Studio, Kotlin, SQL]
category: 'coding'
draft: false
lang: 'en'
---
# 09 competition
:::warning
methods mentioned in the notes should NOT be used for real-life usage softwares
:::
## section 1 - SA&DB Design
### excel
#### XLOOKUP
```
=XLOOKUP(keyword, arrayToSearch, arrayToReturn)
```
<details>
  <summary>senario</summary>
swapping name in A table to its id, where you can find on B table
    
A table:
| Column A |
| -------- |
| company name     |

B table:
| Column A | Column B |
| -------- | -------- |
| company id     | company name     |

in table A do:
`=XLOOKUP(A1,TableB!B2:B100,TableB!A2:A100)`

add `$` in a format like `$B$2:$B$100` for absolute location
</details>

#### randomize data from other table/an array
<details>
<summary>senario</summary>
senario:
generate a list randomize company id

A table:
| Column A |
| -------- |
|company id|

`=INDEX(tableA!A:A,RANDBETWEEN(2,100))`

for an array:
`=INDEX({"hello","bye"},RANDBETWEEN(1,2))`
</details>

#### randomized datetime
dates:`=RANDBETWEEN(DATE(2023,1,1),DATE(2023,12,31))`and change it to datetime format

date + time:`=RANDBETWEEN(DATE(2023,1,1),DATE(2023,12,31)) + RAND()`

**later than previously generated time:**
| Column A | Column B |
| -------- | -------- |
|Start date| End date |

`=A1 + (RAND()*10)`

change `10` according to how far away from start date you want it
#### substitute
![image](src/assets/images/excelSubstitute.png)
#### search
`search("TExt",theCellToSearch)`

returns text position
#### guid
data > from table/range

new column > custom
```
try error "" otherwise Text.NewGuid()
```
### ssms
#### common queries
get data from tables:
`SELECT * FROM db.table`
add new row
`INSERT INTO db.table (Column1, Column2) VALUES ('hello', 'bye')`
modify existing data
`UPDATE db.table SET Column1 = 'Value' WHERE Column2 = 'value'`
remove rows
`DELETE FROM db.table WHERE Column1 = 'value'`
#### flatten

```sql
;WITH CommentTree AS (
    SELECT
        pc.PostCommentId,
        pc.ReplyPostCommentId,
        pc.PostCommentId AS RootCommentId,
        1 AS Depth
    FROM PostComment pc
    WHERE pc.ReplyPostCommentId IS NULL

    UNION ALL

    SELECT
        c.PostCommentId,
        c.ReplyPostCommentId,
        t.RootCommentId,
        t.Depth + 1
    FROM PostComment c
    INNER JOIN CommentTree t
        ON c.ReplyPostCommentId = t.PostCommentId
)
SELECT *
FROM CommentTree;
```
insert
```sql
INSERT INTO New_PostComment
(
    PostCommentId,
    PostId,
    AccountId,
    CommentContent,
    RootCommentId,
    ParentCommentId,
    CommentLevel,
    ReplyToAccountId,
    CreateTime,
    UpdateTime,
    IsDeleted,
    New_OriginalParentCommentId,
    New_OriginalLevel
)
SELECT
    pc.PostCommentId,
    pc.PostId,
    pc.AccountId,
    pc.CommentContent,
    ct.RootCommentId,
    CASE 
        WHEN ct.Depth = 1 THEN NULL
        ELSE ct.RootCommentId
    END AS ParentCommentId,
    CASE 
        WHEN ct.Depth = 1 THEN 1
        ELSE 2
    END AS CommentLevel,
    rp.AccountId AS ReplyToAccountId,
    pc.CreateTime,
    pc.UpdateTime,
    pc.IsDeleted,
    pc.ReplyPostCommentId,
    ct.Depth
FROM PostComment pc
INNER JOIN CommentTree ct
    ON pc.PostCommentId = ct.PostCommentId
LEFT JOIN PostComment rp
    ON pc.ReplyPostCommentId = rp.PostCommentId;
```
## API Setup
### packages, build
open developer powershell
change to project directory
```bash
cd YOUR_PROJECT_NAME
```
install packages ***change `VERSION_HERE` to the version given!***
```bash
dotnet add package Microsoft.EntityFrameworkCore.Design --version VERSION_HERE
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version VERSION_HERE
dotnet tool install --global dotnet-ef --version VERSION_HERE
```
build
```
dotnet build
```
finally build the database
```
dotnet ef dbcontext scaffold "Server=localhost;Database=資料庫名稱;Trusted_Connection=True;Encrypt=False;" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --force
```
![image](src/assets/images/aspBuildSucceed.png)
this means its done successfully ^^^
### setup
in `appsettings.json` add ConnectionStrings
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "WebDatabase": "Server=伺服器;Database=資料庫;Trusted_Connection=True;TrustServerCertificate=true"
  }
}
```
in `Program.cs` add this
change `<WebContext>` to your context in Models folder
```csharp
builder.Services.AddDbContext<WebContext>(options =>
options.UseSqlServer(builder.Configuration.GetConnectionString("WebDatabase")));
```

head to Controllers folder and add a new controller
change accordingly
```csharp
namespace YourProject.Controllers
{
    [Route("api")]
    [ApiController]
    public class MainController : Controller
    {
        private readonly YourContext _YourContext;
        public MainController(YourContext YourContext)
        {
            _YourContext = YourContext;
        }
        
        //start coding!
    }
}
```
Examples:
```csharp
//HTTPGET
[HttpGet("test")]
public IEnumerable<Account> GetAccount()
{
    return _WmlSdAnmContext.Accounts;
}
//HTTPPOST
public class AccountUpdateReq
{
    public string username { get; set; }
    public string result{ get; set; }
}
[HttpPost("admin/action")]
public IActionResult statusAction([FromBody] AccountUpdateReq data)
{
    //.....
    return Ok();
}
```

lastly, code a testing HttpGet
head to Properties folder, launchSettings.json
make sure `"launchBrowser"` is set to true
![image](src/assets/images/launchBrowserTrue.png)
launch it and you should see a website in your browser, add your route after localhost:port/,
then there should be your data listed as json

try commenting/removing those if you cant get into website after publishing
![image](src/assets/images/aspAuthorization.png)
## section 2 - Software Design
### project setting
#### c# version
unload project, open project.csproj
in `<PropertyGroup>`
paste in:
`<langVersion>latest</langVersion>`
### api
#### setup
```csharp
public static HttpClient YourAPI = new HttpClient()
{
    BaseAddress = new Uri("http://10.168.176.36:5000/api/"),
    Timeout = TimeSpan.FromSeconds(20)
};
```
#### get data, deserialize json
```csharp
var response = await YourAPI.GetAsync("route");
string json = await response.Content.ReadAsStringAsync();
var options = new System.Text.Json.JsonSerializerOptions { PropertyNameCaseInsensitive = true };
admins = System.Text.Json.JsonSerializer.Deserialize<List<Admin>>(json, options);
```
#### post data, serialize json
```csharp
var payload = new
{
    name = newMerchantName.Text,
    person = newContactPerson.Text,
    phone = newContactPhone.Text,
    address = newAddress.Text,
    createdBy = Program.currentAdmin.adminID,
    createdAt = DateTime.Now
};
string jsonString = System.Text.Json.JsonSerializer.Serialize(payload);
var content = new StringContent(jsonString, System.Text.Encoding.UTF8, "application/json");
var response = await Program.YourAPI.PostAsync("route", content);
if(response.IsSuccessStatusCode){
    //oh yeah its succes hahahhaha
}
```
### datagridview

:::important
**load, filtering data**
always do it in these steps:
load data(if not loaded) ->
capture filters -> filter data into a new datatable(Task.Run) -> apply to datagridview

**others**
if data is huge, change AutoSizeMode to DisplayedCells
:::
#### filtering huge data
use `await Task.Run`!
```csharp
var rolefilter = RoleFilter.Text;
var search = AdminSearch.Text;
DataTable filteredData = await Task.Run(() =>
{
    var data = admindt.AsEnumerable();
    if (rolefilter != "All" && rolefilter != null && rolefilter != "")
    {
        data = data.Where(x => x.Field<string>("Role") == rolefilter);
    }
    if (search!="" && !search.IsWhiteSpace() && search!=null)
    {
        data = data.Where(x => x.Field<string>("Username").Contains(search) || x.Field<string>("Email").Contains(search));
    }

    if (data.Count() == 0) return null;
    return data.CopyToDataTable();
});
AdminAccountsDataGrid.DataSource = filteredData;
```
#### comboBoxColumn
:::Important
make it not read only (turn off read only in properties)
:::
enable edit with single click
```csharp
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    if (e.RowIndex >= 0 && e.ColumnIndex == dataGridView1.Columns["comboColumn"].Index)
    {
        dataGridView1.BeginEdit(true);
        if (dataGridView1.EditingControl is ComboBox combo)
        {
            combo.DroppedDown = true;
        }
    }
}

private void dataGridView1_CurrentCellDirtyStateChanged(object sender, EventArgs e)
{
    if (dataGridView1.IsCurrentCellDirty)
    {
        dataGridView1.CommitEdit(DataGridViewDataErrorContexts.Commit);
    }
}
```
#### cellpainting
##### button
```csharp
private void dataGridView1_CellPainting(object sender, DataGridViewCellPaintingEventArgs e)
{
    if (e.RowIndex >= 0 && e.ColumnIndex == dataGridView1.Columns["ButtonColumn"].Index)
    {
        e.PaintBackground(e.CellBounds, true);
        // the area which will be drawn, draw selectionbackcolor
        var buttonR = e.CellBounds; //button rectangle
        buttonR.Inflate(-10, -4); //make it smaller (width,height)
        using (Brush brush = new SolidBrush(Color.White))
        {
            e.Graphics.FillRectangle(brush, buttonR);
        }
        //draw border
        using (Pen pen = new Pen(Color.Black,1)) 
        {
            e.Graphics.DrawRectangle(pen, buttonR)
        }
        //draw text
        TextRenderer.DrawText(e.Graphics, "click pls", e.CellStyle.Font, buttonR, Color.White, TextFormatFlags.HorizontalCenter | TextFormatFlags.VerticalCenter);
        //e.graphics, text, font, bounds, color, alignment
        e.Handled = true;
    }
}
```
### dragging form with custom topbar
```csharp
[DllImport("user32.dll")]
public static extern int SendMessage(IntPtr hWnd, int Msg, int wParam, int lParam);
[DllImport("user32.dll")]
public static extern bool ReleaseCapture();
private void header_MouseDown(object sender, MouseEventArgs e)
{
    if(e.Button == MouseButtons.Left)
    {
        ReleaseCapture();
        SendMessage(Handle, 0xA1, 0x2, 0);
    }
}
```
### icon dragging
```csharp
Point mouseOffset;
bool isdDragging = false;

private void icon_MouseDown(object sender, MouseEventArgs e)
{
    if (e.Button == MouseButtons.Left)
    {
        isdDragging = true;
        mouseOffset = new Point(e.X, e.Y);
    }
}
private void icon_MouseUp(object sender, MouseEventArgs e)
{
    isdDragging = false;
}
private void icon_MouseMove(object sender, MouseEventArgs e)
{
    if (isdDragging)
    {
        Point newLocation = icon.Location;
        newLocation.X += e.X - mouseOffset.X;
        newLocation.Y += e.Y - mouseOffset.Y;

        int maxX = this.Width - icon.Width;
        int maxY = this.Height - footer.Height - icon.Height;
        newLocation.X = Math.Max(0, Math.Min(newLocation.X, maxX));
        newLocation.Y = Math.Max(header.Height, Math.Min(newLocation.Y, maxY));

        icon.Location = newLocation;
    }
}
```
### enum
basic usage
```csharp
public enum OrderStatus
{
    Pending,   // Defaults to 0
    Processing, // Defaults to 1
    Shipped,    // Defaults to 2
    Delivered   // Defaults to 3
}
```
```csharp
OrderStatus currentStatus = OrderStatus.Pending;

if (currentStatus == OrderStatus.Pending)
{
    Console.WriteLine("The order is waiting.");
}
```
casting
```csharp
// Enum to Int
int value = (int)OrderStatus.Shipped; // Result: 2

// Int to Enum
OrderStatus status = (OrderStatus)3; // Result: OrderStatus.Delivered
```
### exporting files
#### print to PDF
```csharp
private void exportPDF_Click(object sender, EventArgs e)
{
    //panel indicates the area you want to print out
    Bitmap bmp = new Bitmap(panel.Width, panel.Height);
    panel.DrawToBitmap(bmp, new Rectangle(new Point(), panel.Size));

    using(SaveFileDialog dialog = new SaveFileDialog()
    {
        FileName = "Output.pdf",
        Filter = "PDF檔案 | *.pdf"
    })
    {
        if(dialog.ShowDialog() == DialogResult.OK)
        {
            var pd = new PrintDocument();
            pd.PrinterSettings.PrintToFile = true;
            pd.PrinterSettings.PrintFileName = dialog.FileName;
            pd.PrintController = new StandardPrintController();
            pd.PrintPage += (ss, ee) =>
            {
                //ee.MarginBounds is the A4 paper in pdf's bound
                float ratio = (float)ee.MarginBounds.Width / bmp.Width;
                int newWidth = ee.MarginBounds.Width;
                int newHeight = (int)(bmp.Height * ratio);
                ee.Graphics.DrawImage(bmp, ee.MarginBounds.Left, ee.MarginBounds.Top, newWidth, newHeight);
            };
            pd.Print();
        }
    }
}
```
#### import to csv
```csharp
SaveFileDialog dialog = new SaveFileDialog(){
    Filter = "CSV檔案 | *.csv",
    RestoreDirectory = true,
    FileName = "CSV.csv"
};

if(dialog.ShowDialog() == DialogResult.OK)
{
    StringBuilder sb = new StringBuilder();
    
    string[] headers = dataGridView1.Columns.Cast<DataGridViewColumn>().Select(x => x.HeaderText).ToArray();
    sb.AppendLine(string.Join(",", headers));
    
    foreach (DataGridViewRow row in dataGridView1.Rows)
    {
        string[] cells = row.Cells.Cast<DataGridViewCell>().Select(x => 
        {
            string val = x.Value?.ToString() ?? "";
            return val.Contains(",") ? $"\"{val}\"" : val;
        }).ToArray();
        sb.AppendLine(string.Join(",", cells))
    }
}
File.WriteAllText(dialog.FileName, sb.ToString(), Encoding.UTF8);
```
#### import to json
```csharp
var data = new 
{
    ID = thing.ID,
    Name = thing.Name
};

SaveFileDialog saveJson = new SaveFileDialog()
{
    Filter = "JSON檔案 | *.json",
    FileName = "JSON.json"
};

if(saveJson.ShowDialog() == DialogResult.OK) 
{
    var options = new System.Text.Json.JsonSerializerOptions { WriteIndented = true};
    string jsonString = System.Text.Json.JsonSerializer.Serialize(data, options);
    System.IO.File.WriteAllText(saveJson.FileName, jsonString);
}
```
### importing from file
#### txt from a directory
```csharp
using System.IO;

// ... 在按鈕事件中 ...

string folderPath = @"C:\YourChatLogs"; // 資料夾路徑

if (Directory.Exists(folderPath))
{
    // 取得所有 .txt 檔案的路徑
    string[] filePaths = Directory.GetFiles(folderPath, "*.txt");

    foreach (string path in filePaths)
    {
        string fileName = Path.GetFileName(path);
        string content = File.ReadAllText(path);
    }
}
else
{
    MessageBox.Show("找不到指定的資料夾！");
}
```
#### relative path
```csharp
// 1. 取得目前執行檔的位置 (例如：C:\MyApp\bin\Debug\)
string baseDirectory = AppDomain.CurrentDomain.BaseDirectory;
// 2. 🌟 定義相對路徑：退兩步 (..\..\)，然後進入 Data 找檔案
string relativePath = @"..\..\Data\settings.txt";
// 3. 把路徑拼起來
string combinedPath = Path.Combine(baseDirectory, relativePath);
```
can use `Path.GetFullPath() to clean up the .. in your string`
### paint a circle
#### label
```csharp
private void label1_Paint(object sender, PaintEventArgs e)
{
    e.Graphics.SmoothingMode = SmoothingMode.AntiAlias;

    int circleSize = 12;
    int yPos = (label1.Height - circleSize) / 2;
    //middle
    e.Graphics.FillEllipse(Brushes.Blue, 2, yPos, circleSize, circleSize);
    //color, x, y, width, height
}
```
#### combobox
DrawMode -> OwnerDrawFixed
DropDownStyle -> DropDownList (recommended)
```csharp
private void comboBox1_DrawItem(object sender, DrawItemEventArgs e)
{
    if (e.Index < 0) return;
    ComboBox box = (ComboBox)sender;
    string blue = "#1D10AD";
    Color color = ColorTranslator.FromHtml(blue); // or Color.FromArgb

    e.DrawBackground();
    e.Graphics.SmoothingMode = SmoothingMode.AntiAlias;

    int circleSize = 14;
    Rectangle circleRect = new Rectangle(
        e.Bounds.X + 3,
        e.Bounds.Y + (e.Bounds.Height - circleSize)/2,
        circleSize,
        circleSize
        );//x,y,width,height
    using(SolidBrush brush = new SolidBrush(color))
    {
        e.Graphics.FillEllipse(brush, circleRect);
    }

    //text
    Rectangle textRect = new Rectangle(
        e.Bounds.X + 22,
        e.Bounds.Y,
        e.Bounds.Width - 22,
        e.Bounds.Height
        );
    using(Brush textBrush = new SolidBrush(e.ForeColor))
    {
        StringFormat format = new StringFormat();
        format.LineAlignment = StringAlignment.Center;
        e.Graphics.DrawString(box.Items[e.Index].ToString(), e.Font, textBrush, textRect, format);
    }
    e.DrawFocusRectangle();
}
```
### sha256
```csharp
public static string ToSHA256(string input)
{
    using (SHA256 sha256 = SHA256.Create())
    {
        byte[] clearValue = Encoding.UTF8.GetBytes(input);
        byte[] hashedValue = sha256.ComputeHash(clearValue);
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < hashedValue.Length; i++)
        {
            builder.Append(hashedValue[i].ToString("x2"));
        }
        string hashedString = builder.ToString();
   
        return hashedString;
    }
}
```
### user control
#### loadrooms
```csharp
public void loadRooms()
        {
            var roomsids = Program.chatRoomMembers.Where(m => m.accountId == Program.loggedacc.accountId).Select(m => m.chatRoomId).ToList();
            var rooms = Program.chatRooms.Where(r => roomsids.Contains(r.chatRoomId)).ToList();
            chatRooms.Controls.Clear();

            foreach(var room in rooms)
            {
                var item = new RoomItem(room.chatRoomId, room.chatRoomName, room.lastMessageDateTime);

                item.RoomSelected += (s, roomId) =>
                {
                    switchRoom(roomId);
                };
                chatRooms.Controls.Add(item);
            }
        }

```
#### rooms
```csharp
public int roomId { get; set; }

        public event EventHandler<int> RoomSelected;

        public RoomItem(int id,string rname, DateTime? lastMessageTime)
        {
            InitializeComponent();
            this.roomId = id;
            name.Text = rname;
            time.Text = lastMessageTime == null? "(空的聊天室)":((DateTime)lastMessageTime).ToString("M/d HH:mm");

            this.Click += (s, e) => RoomSelected?.Invoke(this, this.roomId);
            name.Click += (s, e) => RoomSelected?.Invoke(this, this.roomId);
            time.Click += (s, e) => RoomSelected?.Invoke(this, this.roomId);
        }
```
#### messages
```csharp
public void SetMessage(string message,string userandtime, bool isMine, string type, bool isViolation)
        {
            chatmessage.Text = message == null? $"({type})":message;
            UserAndTime.Text = userandtime;
            if (isViolation)
            {
                chatmessage.ForeColor = Color.DarkRed;
                chatmessage.Font = new Font(chatmessage.Font, FontStyle.Italic);
            }
            if (isMine)
            {
                UserAndTime.Location = new Point(this.Width-UserAndTime.Width, 11);
                chatmessage.Location = new Point(this.Width-chatmessage.Width, 29);
            }
            else
            {
                UserAndTime.Location = new Point(3, 11);
                chatmessage.Location = new Point(3, 29);
            }
            this.Height += chatmessage.Height - 18;
        }
```
## section 3 - App Design
### versions
![image](https://hackmd.io/_uploads/HJcholrq-g.png)
### viewmodel
#### normal
```kotlin
class MainViewModel : ViewModel() {
    // 外部可以讀取，但只有 ViewModel 內部可以修改
    var page by mutableStateOf(1)
        private set 

    fun changePage(newPage: Int) {
        // 可以在這裡加上檢查邏輯，例如 page 不得小於 1
        if (newPage > 0) {
            page = newPage
        }
    }
}
```
#### stateflow
```kotlin
class MyViewModel : ViewModel() {
    // 1. 私有的 MutableStateFlow，用來更新資料
    private val _userCount = MutableStateFlow(0)
    
    // 2. 公開的唯讀 StateFlow，讓 Activity 觀察
    val userCount: StateFlow<Int> = _userCount

    // 3. 處理業務邏輯
    fun incrementCount() {
        _userCount.value += 1
    }
}
class MainActivity : AppCompatActivity() {
    // 使用 viewModels 委託來取得實例
    private val viewModel: MyViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 觀察資料變化並更新 UI
        lifecycleScope.launch {
            viewModel.userCount.collect { count ->
                binding.textView.text = "Count: $count"
            }
        }

        binding.button.setOnClickListener {
            viewModel.incrementCount()
        }
    }
}
```
### sharedpreferences
```kotlin
// for "remember me" login
object TokenManager{
    private const val pref_name = "mySecurePrefs"
    private const val key = "user_token"
    
    fun saveToken(context: Context, token: String){
        val prefs: SharedPreferences = context.getSharedPreferences(pref_name, Context.MODE_PRIVATE)
        val editor = prefs.edit()
        editor.putString(key, token)
        editor.apply
    }
    fun getToken(context: Context): String? {
        val prefs: SharedPreferences = context.getSharedPreferences(pref_name, Context.MODE_PRIVATE)
        return prefs.getString(key, null) //return null if not found
    }
    fun clearToken(context: Context) {
        val prefs: SharedPreferences = context.getSharedPreferences(pref_name, Context.MODE_PRIVATE)
        val editor = prefs.edit()
        editor.remove(key)
        editor.apply()
    }
}
```
### api
add those in AndroidManifest.xml
![image](https://hackmd.io/_uploads/HJsbikHcbx.png)

```kotlin
suspend fun httpGet(urlString: String): String = withContext(Dispatchers.IO){
    var connection: HttpURLConnection? = null
    return@withContext try {
        val url = URL("http://localhost:80/api/${urlString}")
        connection = (url.openConnection() as HttpURLConnection).apply {
            requestMethod = "GET"
            connectTimeout = 8000
            readTimeout = 8000
        }
        val code = connection.responseCode
        val stream = if(code in 200..299) connection.inputStream else connection.errorStream
        val body = stream?.let {
            it.bufferedReader().use {it.readText()}}
        } ?: "No response body"
        
        "HTTP $code $body"
    } catch (ex: Exception) {
        Log.e("ApiError", "failed to http get", ex)
    } finally {
        connection?.disconnect()
    }
}
suspend fun httpPost(urlString: String, jsonPayload): String = withContext(Dispatchers.IO) {
    var connection: HttpURLConnection? = null
    return@withContext try {
        val url = URL("http://localhost:80/api/${urlString}")
        connection = (url.openConnection() as HttpURLConnection).apply {
            requestMethod = "POST"
            connectTimeout = 5000
            setRequestProperty("Content-Type", "application/json; charset=UTF-8")
            setRequestProperty("Accept", "application/json")
            doOutput = true
        }
        
        connection.outputStream.bufferedWriter().use {
            it.write(jsonPayload)
        }
        
        val code = connection.responseCode
        val stream = if (code in 200..299) connection.inputStream else connection.errorStream
        val body = stream?.let {
            it.bufferedReader().use {it.readText()}
        }? : "No response body"
        
        "HTTP $code $body"
    } catch(ex: Exception) {
        Log.e("ApiError", "failed to http post", ex)
    } finally{
        connection?.disconnect()
    }
}
```
fetch data
```kotlin
private suspend fun fetchDataFromApi(): List<Data> {
    val resultList = mutableListOf<Data>()
    try{
        val result = httpGet("data")
        
        if(result.startsWith("HTTP 200")) {
            val jsonStartIndex = result.indexOfFirst{it == '[' || it == '{'}
            if(jsonStartIndex != -1) {
                val jsonString = result.substring(jsonStartIndex)
                val jsonArray = JSONArray(jsonString)
                
                for(i in 0 until jsonArray.length()){
                    val jsonObject = jsonArray.getJSONObject(i)
                    
                    resultList.add(Data(
                        dataId = jsonObject.optInt("dataId", -1),
                        dataName = jsonObject.optString("dataName", "UNKNOWN_DATA")
                        dataIsActive = jsonObject.optBoolean("isActive", false)
                        //...
                    ))
                }
            }
        }
    } catch(ex: Exception){
        Log.e("ApiFetchData", "failed to fetch data from api", ex)
    }
    return resultList
}
```
### LINQ
![image](https://hackmd.io/_uploads/rkqol5Pcbg.png)
add `.asSequence()` for huge datas
### PullToRefresh
```kotlin
val refreshState = rememberPullToRefreshState()
Box(
    modifier = Modifier.nestedScroll(refreshState.nestedScrollConnection))
{
    PullToRefreshContainer(
            modifier = Modifier.align(Alignment.TopCenter),
            state = refreshState
        )
        if(refreshState.isRefreshing){
            LaunchedEffect(true) {
                vm.fetchFriends()
                vm.fetchAccounts()
                vm.fetchFriendRequests()
                refreshState.endRefresh()
            }
        }
}
```
## Keymap
- Global
    - `Ctrl + S` save all
    - `Ctrl + D` duplicate current line
    - `Alt + drag` edit multiple lines
- Visual Studio
    - `Ctrl + Shift + A` create new item
    - `Ctrl + Alt + X` toolbox
    - `Ctrl + Alt + T` items browser
    - `F7` switch to form.cs
    - `Ctrl + F7` switch to form design
    - `Ctrl + Tab` switch between opened files
- SSMS
    - `F5` execute query
- Android Studio
    - `Ctrl + Alt + L` format
## packages!!
important:
```
dotnet-ef 9.0.13

EntityFramework 6.5.1

Microsoft.EntityFrameworkCore 9.0.13

Microsoft.EntityFrameworkCore.Design 9.0.13

Microsoft.EntityFrameworkCore.SqlServer 9.0.13

System.Text.Json 9.0.13
```
all:
```
Azure.Core 1.38.0

Azure.Identity 1.11.4

dotnet-ef 9.0.13

EntityFramework 6.5.1

Humanizer.Core 2.14.1

Microsoft.Bcl.AsyncInterfaces 1.1.1

Microsoft.Bcl.AsyncInterfaces 7.0.0

Microsoft.Build.Framework 16.10.0

Microsoft.Build.Framework 17.8.43

Microsoft.Build.Locator 1.7.8

Microsoft.CodeAnalysis.Analyzers 3.3.4

Microsoft.CodeAnalysis.Common 4.8.0

Microsoft.CodeAnalysis.CSharp 4.8.0

Microsoft.CodeAnalysis.CSharp.Workspaces 4.8.0

Microsoft.CodeAnalysis.Workspaces.Common 4.8.0

Microsoft.CodeAnalysis.Workspaces.MSBuild 4.8.0

Microsoft.CSharp 4.5.0

Microsoft.Data.SqlClient 5.1.6

Microsoft.Data.SqlClient.SNI.runtime 5.1.1

Microsoft.EntityFrameworkCore 9.0.13

Microsoft.EntityFrameworkCore.Abstractions 9.0.13

Microsoft.EntityFrameworkCore.Analyzers 9.0.13

Microsoft.EntityFrameworkCore.Design 9.0.13

Microsoft.EntityFrameworkCore.Relational 9.0.13

Microsoft.EntityFrameworkCore.SqlServer 9.0.13

Microsoft.Extensions.ApiDescription.Server 9.0.0

Microsoft.Extensions.Caching.Abstractions 9.0.13

Microsoft.Extensions.Caching.Memory 9.0.13

Microsoft.Extensions.Configuration.Abstractions 9.0.13

Microsoft.Extensions.DependencyInjection 9.0.13

Microsoft.Extensions.DependencyInjection.Abstractions 9.0.13

Microsoft.Extensions.DependencyModel 9.0.13

Microsoft.Extensions.Logging 9.0.13

Microsoft.Extensions.Logging.Abstractions 9.0.13

Microsoft.Extensions.Options 9.0.13

Microsoft.Extensions.Primitives 9.0.13

Microsoft.Identity.Client 4.61.3

Microsoft.Identity.Client.Extensions.Msal 4.61.3

Microsoft.IdentityModel.Abstractions 6.35.0

Microsoft.IdentityModel.JsonWebTokens 6.35.0

Microsoft.IdentityModel.Logging 6.35.0

Microsoft.IdentityModel.Protocols 6.35.0

Microsoft.IdentityModel.Protocols.OpenIdConnect 6.35.0

Microsoft.IdentityModel.Tokens 6.35.0

Microsoft.NETCore.Platforms 1.1.0

Microsoft.NETCore.Targets 1.1.0

Microsoft.OpenApi 1.6.25

Microsoft.SqlServer.Server 1.0.0

Microsoft.Win32.SystemEvents 6.0.0

Mono.TextTemplating 3.0.0

Swashbuckle.AspNetCore 9.0.6

Swashbuckle.AspNetCore.Swagger 9.0.6

Swashbuckle.AspNetCore.SwaggerGen 9.0.6

Swashbuckle.AspNetCore.SwaggerUI 9.0.6

System.ClientModel 1.0.0

System.CodeDom 6.0.0

System.Collections.Immutable 7.0.0

System.Composition 7.0.0

System.Composition.AttributedModel 7.0.0

System.Composition.Convention 7.0.0

System.Composition.Hosting 7.0.0

System.Composition.Runtime 7.0.0

System.Composition.TypedParts 7.0.0

System.Configuration.ConfigurationManager 6.0.0

System.Configuration.ConfigurationManager 6.0.1

System.Diagnostics.DiagnosticSource 6.0.1

System.Drawing.Common 6.0.0

System.Formats.Asn1 5.0.0

System.Formats.Asn1 9.0.13

System.IdentityModel.Tokens.Jwt 6.35.0

System.IO.Pipelines 7.0.0

System.Memory 4.5.4

System.Memory.Data 1.0.2

System.Numerics.Vectors 4.5.0

System.Reflection.Metadata 7.0.0

System.Runtime 4.3.0

System.Runtime.Caching 6.0.0

System.Runtime.CompilerServices.Unsafe 6.0.0

System.Security.AccessControl 6.0.0

System.Security.Cryptography.Cng 4.5.0

System.Security.Cryptography.Cng 5.0.0

System.Security.Cryptography.ProtectedData 4.5.0

System.Security.Cryptography.ProtectedData 4.7.0

System.Security.Cryptography.ProtectedData 6.0.0

System.Security.Permissions 6.0.0

System.Security.Principal.Windows 5.0.0

System.Text.Encoding 4.3.0

System.Text.Encoding.CodePages 6.0.0

System.Text.Encodings.Web 4.7.2

System.Text.Encodings.Web 6.0.0

System.Text.Json 4.6.0

System.Text.Json 4.7.2

System.Text.Json 7.0.3

System.Text.Json 9.0.13

System.Threading.Channels 7.0.0

System.Threading.Tasks.Extensions 4.5.4

System.Windows.Extensions 6.0.0
```