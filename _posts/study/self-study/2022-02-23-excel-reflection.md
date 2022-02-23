---
title : "Spring Apache POI를 활용한 엑셀 파일 생성/다운로드 쉽게하기"
categories : 
    - spring
tag :
    - spring
toc : true
---

#### **환경**
Dependency는 poi-ooxml 5.0.0을 사용했다.  
또한 원한다면 FileOutputStream을 통해서 다운로드 받을 수 있지만  
브라우저상에서 다운로드 되는 것을 보고 싶었기 때문에 servlet-api 2.5도 추가했다.  

구현된 전체 소스는 깃허브에 올려두었다.  
[Excel-Maker 소스보기](https://github.com/LeeGiCheol/excel-maker)

---

#### **만들게 된 계기**

Apache POI를 사용하여 엑셀 파일을 생성하는 라이브러리를 소개하고자 한다.  

2020년 신입으로 회사에 입사 후 6개월차에  
생각보다 굵직한 업무를 맡게되었다.  

사용량은 많지 않지만 뿔뿔이 흩어져있는 레거시 API 코드를 통합할 수 있는 API를 만드는 것이었다.  

말은 거창하지만 사실 API를 만드는데에는 큰 문제가 없었고, 오히려 정산이 문제였다.  

기존 정산 페이지는 고객사별로 사용량을 엑셀로 다운받을 수 있도록 되어있는데  
Apache POI 라이브러리를 통해 구현되어있었다.  

문제는 고객사마다 원하는 엑셀 레이아웃이 조금씩 달랐고,  
그로인해 엑셀을 다운받을 수 있는 메서드는 고객사별로 제각각이었다.  

POI는 구현하기 까다롭고 (한 메서드에 250 라인 정도 였다.)  
메서드를 통합하기도 까다로워 보였다.  

마침 [백기선님의 더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation) 강의를 보고 있었는데  
강의 내용 중 하나인 리플렉션을 활용하기 딱 좋은 소스라고 생각되어 개발하게 되었다.  

---

#### **아파치 POI 공식 홈페이지의 예제 코드**

먼저 아파치 POI 공식 홈페이지의 예제 코드이다.  

솔직히 예제 코드도 결과도 친절하진 않은 것 같다..  
(예제와 버전이 다른지 실행되지 않는 라인도 있음)  

<details>
<summary><b>코드 보기</b></summary>
<div markdown="1">

```java
public static void main(String[] args) {
    short rownum;
    // create a new file
    FileOutputStream out = new FileOutputStream("workbook.xls");
    // create a new workbook
    Workbook wb = new HSSFWorkbook();
    // create a new sheet
    Sheet s = wb.createSheet();
    // declare a row object reference
    Row r = null;
    // declare a cell object reference
    Cell c = null;
    // create 3 cell styles
    CellStyle cs = wb.createCellStyle();
    CellStyle cs2 = wb.createCellStyle();
    CellStyle cs3 = wb.createCellStyle();
    DataFormat df = wb.createDataFormat();
    // create 2 fonts objects
    Font f = wb.createFont();
    Font f2 = wb.createFont();
    //set font 1 to 12 point type
    f.setFontHeightInPoints((short) 12);
    //make it blue
    f.setColor( (short)0xc );
    // make it bold
    //arial is the default font
    f.setBoldweight(Font.BOLDWEIGHT_BOLD);
    //set font 2 to 10 point type
    f2.setFontHeightInPoints((short) 10);
    //make it red
    f2.setColor( (short)Font.COLOR_RED );
    //make it bold
    f2.setBoldweight(Font.BOLDWEIGHT_BOLD);
    f2.setStrikeout( true );
    //set cell stlye
    cs.setFont(f);
    //set the cell format
    cs.setDataFormat(df.getFormat("#,##0.0"));
    //set a thin border
    cs2.setBorderBottom(cs2.BORDER_THIN);
    //fill w fg fill color
    cs2.setFillPattern((short) CellStyle.SOLID_FOREGROUND);
    //set the cell format to text see DataFormat for a full list
    cs2.setDataFormat(HSSFDataFormat.getBuiltinFormat("text"));
    // set the font
    cs2.setFont(f2);
    // set the sheet name in Unicode
    wb.setSheetName(0, "\u0422\u0435\u0441\u0442\u043E\u0432\u0430\u044F " +
                    "\u0421\u0442\u0440\u0430\u043D\u0438\u0447\u043A\u0430" );
    // in case of plain ascii
    // wb.setSheetName(0, "HSSF Test");
    // create a sheet with 30 rows (0-29)
    int rownum;
    for (rownum = (short) 0; rownum < 30; rownum++)
    {
        // create a row
        r = s.createRow(rownum);
        // on every other row
        if ((rownum % 2) == 0)
        {
            // make the row height bigger  (in twips - 1/20 of a point)
            r.setHeight((short) 0x249);
        }
        //r.setRowNum(( short ) rownum);
        // create 10 cells (0-9) (the += 2 becomes apparent later
        for (short cellnum = (short) 0; cellnum < 10; cellnum += 2)
        {
            // create a numeric cell
            c = r.createCell(cellnum);
            // do some goofy math to demonstrate decimals
            c.setCellValue(rownum * 10000 + cellnum
                    + (((double) rownum / 1000)
                    + ((double) cellnum / 10000)));
            String cellValue;
            // create a string cell (see why += 2 in the
            c = r.createCell((short) (cellnum + 1));
            // on every other row
            if ((rownum % 2) == 0)
            {
                // set this cell to the first cell style we defined
                c.setCellStyle(cs);
                // set the cell's string value to "Test"
                c.setCellValue( "Test" );
            }
            else
            {
                c.setCellStyle(cs2);
                // set the cell's string value to "\u0422\u0435\u0441\u0442"
                c.setCellValue( "\u0422\u0435\u0441\u0442" );
            }
            // make this column a bit wider
            s.setColumnWidth((short) (cellnum + 1), (short) ((50 * 8) / ((double) 1 / 20)));
        }
    }
    //draw a thick black border on the row at the bottom using BLANKS
    // advance 2 rows
    rownum++;
    rownum++;
    r = s.createRow(rownum);
    // define the third style to be the default
    // except with a thick black border at the bottom
    cs3.setBorderBottom(cs3.BORDER_THICK);
    //create 50 cells
    for (short cellnum = (short) 0; cellnum < 50; cellnum++)
    {
        //create a blank type cell (no value)
        c = r.createCell(cellnum);
        // set it to the thick black border style
        c.setCellStyle(cs3);
    }
    //end draw thick black border
    // demonstrate adding/naming and deleting a sheet
    // create a sheet, set its title then delete it
    s = wb.createSheet();
    wb.setSheetName(1, "DeletedSheet");
    wb.removeSheetAt(1);
    //end deleted sheet
    // write the workbook to the output stream
    // close our file (don't blow out our file handles
    wb.write(out);
    out.close();
}
```
</div>
</details>  

<br>

![error](/assets/images/self-study/2022-02-23/apache-poi-exam.png)  

---

### **ExcelMaker**

#### **클라이언트 구현 예시**  
아래는 클라이언트에서 사용하는 경우이다.  

```java
@RestController
public class ExcelDown {

    @PostMapping("/excel/download")
    public void excelDownload(HttpServletResponse response) {
        List<Users> userList = getUsers(10);

        int columnSize = 1;

        ExcelMaker excelMaker = new ExcelMaker();
        excelMaker.setSheetName("AAAAA")
                .setFileExtension("xlsx")
                .setRemoveField("id")
                .setChangeFieldName("name", "이름")
                .setChangeFieldName("phoneNumber", "핸드폰 번호")
                .makeExcel(response, Users.class, userList, columnSize);
    }

    private List<User> getUsers(int index) {
        List<User> userList = new ArrayList<>();
        for (long i = 0; i < index; i++) {
            Users users = new Users();
            users.setId(i);
            users.setName("LEE" + i);
            users.setPhoneNumber("010-xxxx-xxx" + i);
            userList.add(users);
        }
        return userList;
    }

}


public class Users {

    private Long id;
    private String name;
    private String phoneNumber;

    // getter, setter...

}
```

공식 문서에 비해 훨씬 간단하게 엑셀 다운로드가 가능하다.  
기본 세팅이 되어있는 메서드를 제외하면  
사실상 makeExcel 메서드 하나만으로 엑셀을 만들 수 있다.  

![error](/assets/images/self-study/2022-02-23/excel-maker.png)  

---

#### **내부 구현 코드**  

##### **Dependency**  
앞서 말한 듯 servlet-api와 poi-ooxml을 사용하였다.  

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <scope>provided</scope>
    <version>2.5</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.0.0</version>
</dependency>
```

---

##### **필드**  

구현에 사용된 필드는 아래와 같다.  
```java
/**
 * 파일, 시트 이름
 */
private String sheetName = "download";

/**
 * 확장자명
 */
private String fileExtension = ".xlsx";

/**
 * 사용하지 않는 필드명
 */
private String removeField;

/**
 * 사용하지 않는 필드 리스트
 */
private List<String> removeFields = new ArrayList<>();

/**
 * 엑셀에 표기될 필드 이름
 */
private final Map<String, String> changeFieldName = new HashMap<>();
```

<br> 

아래는 필드의 값을 컨트롤 할 수 있는 getter setter이다.  
이를 통해 파일/시트명, 확장자를 변경할 수 있고,  

사용하지 않을 필드를 지정할 수 있다.  

리플렉션 기반이기 때문에 table header에 입력 될 값은  
기본적으로 데이터 클래스의 필드명을 따른다.  
만약 필드명과 다른 이름을 사용하고 싶다면 setChangeFieldName을 사용할 수 있다.  

필드명의 변경이 필요한 경우 클라이언트 코드에서 반복적인 코드가 많기 때문에  
setter는 자기 자신을 리턴함으로써 Method Chaining이 가능하도록 구현했다.  

```java
public ExcelMaker setSheetName(String sheetName) {
    this.sheetName = sheetName;
    return this;
}

public ExcelMaker setFileExtension(String fileExtension) {
    this.fileExtension = "." + fileExtension;
    return this;
}

public ExcelMaker setRemoveField(String removeField) {
    this.removeFields.add(removeField);
    return this;
}

private List<String> getRemoveFields() {
    return removeFields;
}

public ExcelMaker setRemoveFields(List<String> removeFields) {
    this.removeFields = removeFields;
    return this;
}

private String getChangeFieldName(String fieldName) {
    return changeFieldName.get(fieldName);
}

public ExcelMaker setChangeFieldName(String oldFieldName, String newFieldName) {
    this.changeFieldName.put(oldFieldName, newFieldName);
    return this;
}
```

---

##### **상세 로직**  
makeExcel 메서드의 파라미터는  
Response Header를 만들 HttpServletResponse,  
생성할 엑셀에 들어가는 데이터의 클래스와  
데이터 리스트,  
컬럼의 가로 병합 크기 (전체 적용)

총 4개이다.  

(마지막 파라미터인 columnSize를 기입하지 않는다면 기본값인 1을 설정한다.)  

<br>

구현 로직은 크게 6개의 단계로 진행한다.  


1. 먼저 엑셀 파일과, 시트, 행을 만들고, 셀을 선언했다.  
이때 OutputStream를 사용하기 때문에 try 블록이 종료되면  
자동으로 자원 해제를 할 수 있도록 `try-with-resources` 문법을 사용한다.  
아쉽게도 SXSSFWorkbook 객체는 finally를 통해 해제를 해야한다.  

```java
SXSSFWorkbook wb = new SXSSFWorkbook();

try (OutputStream output = response.getOutputStream()) {
    int cellNum = 0;
    int currentRow = 0;
    String colTitle;

    Sheet sh = wb.createSheet(sheetName);
    Row row = sh.createRow(currentRow++);
    Cell cell;
} 
```

<br>

2. 사용하지 않는 필드를 필터링 후 데이터 클래스에서 실제 사용할 필드를 찾는다.    

```java
Field[] allFieldsName = voClass.getDeclaredFields();
List<Field> useFieldName = getUseField(allFieldsName);
```

```java
private List<Field> getUseField(Field[] allFieldsName) {
    List<Field> useFieldName = new ArrayList<>();

    if (getRemoveFields().size() == 0) {
        return Arrays.asList(allFieldsName);
    }

    boolean flag;
    for (Field value : allFieldsName) {
        flag = true;
        for (int i = 0; i < getRemoveFields().size(); i++) {
            if (value.getName().equals(getRemoveFields().get(i))) {
                flag = false;
                break;
            }
        }

        if (flag) {
            useFieldName.add(value);
        }
    }

    return useFieldName;
}
```

<br>

3. makeExcel 파라미터로 받은 columnSize의 크기로 셀을 가로로 병합한다.  

```java
int fieldSize = useFieldName.size();
mergeRowRegion(sh, fieldSize, columnSize);
```

```java
private void mergeRowRegion(Sheet sh, int fieldSize, int columnSize) {
    if (columnSize == 1) {
        return;
    }

    int currentColumn = 0;
    for (int i = 0; i < fieldSize; i++) {
        sh.addMergedRegion(new CellRangeAddress(0, 0, currentColumn, currentColumn + columnSize - 1));
        currentColumn += columnSize;
    }
}
```

<br>

4. 첫 번째 row에 적용 될 table header의 텍스트를 설정한다.  

```java
int fieldNumber = 0;

for (int i = 0; i < fieldSize * columnSize; i+=columnSize) {
    cell = row.createCell(cellNum + i);
    colTitle = useFieldName.get(fieldNumber++).getName();

    if (getChangeFieldName(colTitle) != null) {
        colTitle = getChangeFieldName(colTitle);
    }

    cell.setCellValue(colTitle);
}
```

<br>

5. row에 들어가는 데이터를 설정한다.  

```java
for (Object data : dataList) {
    row = sh.createRow(currentRow++);
    setFieldValues(sh, row, allFieldsName, useFieldName, data, columnSize);
}
```

<br>

필드의 경우 접근제어자가 private인 경우가 대다수이기 때문에  
리플렉션을 사용하기 위해선 field.setAccessible(true) 를 꼭 해줘야 한다.  

Object cellValue는 useFieldName.get(i)를 통해 Field 객체를 리턴받는다.  
이때 Field.get의 파라미터로 해당하는 객체를 넣어주면  
해당 필드의 Value를 받을 수 있다.  

```java
private void setFieldValues(Sheet sh, Row row, Field[] allFieldsName, List<Field> useFieldName, Object data, int columnSize) throws IllegalAccessException {
    Cell cell;
    int fieldNumber = 0;
    int cellnum = 0;

    for (int i = 0; i < useFieldName.size(); i++) {
        if (!allFieldsName[fieldNumber++].equals(useFieldName.get(i))) {
            i--;
            continue;
        }

        useFieldName.get(i).setAccessible(true);

        Object cellValue = useFieldName.get(i).get(data);

        mergeRowRegion(sh, cellnum, columnSize);

        cell = row.createCell(cellnum);
        cell.setCellValue(String.valueOf(cellValue));
        cellnum += columnSize;
    }
}
```

<br>

6. 파일 이름과 Response의 Header를 작성하고 엑셀 파일을 다운로드한다.  

```java
String fileName = this.sheetName;
fileName = new String(fileName.getBytes("UTF-8"), "ISO-8859-1");

response.reset();
response.setContentType("application/octet-stream");
response.setHeader("Content-Disposition", "attachment;filename=\"" + fileName + this.fileExtension + "\"");

wb.write(output);
```

<br>

맨 처음 말했던대로 SXSSFWorkbook 객체는  
직접 finally 구문을 통해 dispose한다.  

Exception 처리는 클라이언트가 직접하는 것이 맞다고 생각해  
catch 구문을 사용하지 않고 throws를 했다.  

```java
} finally {
    wb.dispose();
}
```

<br>

아래는 전체 로직이다.  
```java
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;

import javax.servlet.http.HttpServletResponse;
import java.io.OutputStream;
import java.lang.reflect.Field;
import java.util.*;

/**
 * VO에 저장된 필드명과 해당 VO의 데이터를 통해
 * 엑셀을 쉽게 만든다.
 *
 * @author LEEGICHEOL
 * @since 2021.03.26
 */
public class ExcelMaker {

    /**
     * 파일, 시트 이름
     */
    private String sheetName = "download";

    /**
     * 확장자명
     */
    private String fileExtension = ".xlsx";

    /**
     * 사용하지 않는 필드명
     */
    private String removeField;

    /**
     * 사용하지 않는 필드 리스트
     */
    private List<String> removeFields = new ArrayList<>();

    /**
     * 엑셀에 표기될 필드 이름
     */
    private final Map<String, String> changeFieldName = new HashMap<>();


    public ExcelMaker setSheetName(String sheetName) {
        this.sheetName = sheetName;
        return this;
    }

    public ExcelMaker setFileExtension(String fileExtension) {
        this.fileExtension = "." + fileExtension;
        return this;
    }

    public ExcelMaker setRemoveField(String removeField) {
        this.removeFields.add(removeField);
        return this;
    }

    private List<String> getRemoveFields() {
        return removeFields;
    }

    public ExcelMaker setRemoveFields(List<String> removeFields) {
        this.removeFields = removeFields;
        return this;
    }

    private String getChangeFieldName(String fieldName) {
        return changeFieldName.get(fieldName);
    }

    public ExcelMaker setChangeFieldName(String oldFieldName, String newFieldName) {
        this.changeFieldName.put(oldFieldName, newFieldName);
        return this;
    }

    /**
     * makeExcel 사용시 columnSize를 기입하지 않은 경우 기본 값 1을 설정한다.
     */
    public void makeExcel(HttpServletResponse response, Class<?> voClass, List<?> dataList) throws IOException, IllegalAccessException {
        makeExcel(response, voClass, dataList, 1);
    }


    /**
     * 엑셀을 만든다.
     *
     * @param response    웹사이트 저장 용도
     * @param voClass     VO 클래스
     * @param dataList    엑셀에 표기할 데이터 리스트
     * @param columnSize  컬럼 가로 길이
     * @throws IOException             SXSSFWorkbook.write, HttpServletResponse.getOutputStream, new String(byte[] bytes, String charsetName)
     * @throws IllegalAccessException  setFieldValues
     *
     */
    public void makeExcel(HttpServletResponse response, Class<?> voClass, List<?> dataList, int columnSize) throws IOException, IllegalAccessException {
        SXSSFWorkbook wb = new SXSSFWorkbook();

        try (OutputStream output = response.getOutputStream()) {
            int cellNum = 0;
            int currentRow = 0;
            String colTitle;

            Sheet sh = wb.createSheet(sheetName);
            Row row = sh.createRow(currentRow++);
            Cell cell;

            Field[] allFieldsName = voClass.getDeclaredFields();
            List<Field> useFieldName = getUseField(allFieldsName);

            int fieldSize = useFieldName.size();
            mergeRowRegion(sh, fieldSize, columnSize);

            int fieldNumber = 0;

            for (int i = 0; i < fieldSize * columnSize; i+=columnSize) {
                cell = row.createCell(cellNum + i);
                colTitle = useFieldName.get(fieldNumber++).getName();

                if (getChangeFieldName(colTitle) != null) {
                    colTitle = getChangeFieldName(colTitle);
                }

                cell.setCellValue(colTitle);
            }

            for (Object data : dataList) {
                row = sh.createRow(currentRow++);
                setFieldValues(sh, row, allFieldsName, useFieldName, data, columnSize);
            }

            String fileName = this.sheetName;
            fileName = new String(fileName.getBytes("UTF-8"), "ISO-8859-1");

            response.reset();
            response.setContentType("application/octet-stream");
            response.setHeader("Content-Disposition", "attachment;filename=\"" + fileName + this.fileExtension + "\"");

            wb.write(output);
        } finally {
            wb.dispose();
        }
    }


    /**
     * 컬럼 가로를 columnSize만큼 병합한다.
     *
     * @param sh         시트
     * @param fieldSize  컬럼 개수
     * @param columnSize 컬럼 가로 길이
     */
    private void mergeRowRegion(Sheet sh, int fieldSize, int columnSize) {
        if (columnSize == 1) {
            return;
        }

        int currentColumn = 0;
        for (int i = 0; i < fieldSize; i++) {
            sh.addMergedRegion(new CellRangeAddress(0, 0, currentColumn, currentColumn + columnSize - 1));
            currentColumn += columnSize;
        }
    }


    /**
     * VO 필드 중 실제 사용되는 필드를 찾는다.
     *
     * @param allFieldsName VO의 전체 필드
     * @return              VO 필드 중 사용되는 필드
     */
    private List<Field> getUseField(Field[] allFieldsName) {
        List<Field> useFieldName = new ArrayList<>();

        if (getRemoveFields().size() == 0) {
            return Arrays.asList(allFieldsName);
        }

        boolean flag;
        for (Field value : allFieldsName) {
            flag = true;
            for (int i = 0; i < getRemoveFields().size(); i++) {
                if (value.getName().equals(getRemoveFields().get(i))) {
                    flag = false;
                    break;
                }
            }

            if (flag) {
                useFieldName.add(value);
            }
        }

        return useFieldName;
    }


    /**
     * 데이터를 삽입한다.
     *
     * @param sh                시트
     * @param row               열
     * @param allFieldsName     VO의 전체 필드
     * @param useFieldName      VO 필드 중 사용되는 필드
     * @param data              입력할 데이터
     * @param columnSize        컬럼 가로 길이
     * @throws IllegalAccessException  Object cellValue = useFieldName.get(i).get(data); -> 참조할 수 없는 필드일 경우 Exception 발생
     */
    private void setFieldValues(Sheet sh, Row row, Field[] allFieldsName, List<Field> useFieldName, Object data, int columnSize) throws IllegalAccessException {
        Cell cell;
        int fieldNumber = 0;
        int cellnum = 0;

        for (int i = 0; i < useFieldName.size(); i++) {
            if (!allFieldsName[fieldNumber++].equals(useFieldName.get(i))) {
                i--;
                continue;
            }

            useFieldName.get(i).setAccessible(true);

            Object cellValue = useFieldName.get(i).get(data);

            mergeRowRegion(sh, cellnum, columnSize);

            cell = row.createCell(cellnum);
            cell.setCellValue(String.valueOf(cellValue));
            cellnum += columnSize;
        }
    }

}
```

---

#### **개선이 필요한 사항**
ExcelMaker 클래스는 빌더 패턴을 적용해도 괜찮을 것이라 생각한다.  

1년정도 지난 시점에서 다시 보니  
주석도 많이 달아놨고, 나름 공을 들여 만든 소스라 로직이 대부분 기억나지만  
충분히 리팩토링 할 부분이 많았다.  

getUseField, setFieldValues와 같은 성의없는 메서드명과  
StandardCharsets에 상수로 선언된 인코딩 타입이 있음에도 불구하고 문자열로 입력한 것,  
Sheet에 해당하는 필드와, 필드에 해당하는 필드 등 관련있는 변수를 묶어  
VO 클래스로 만들어 사용해도 좋을 것 같다.  

특히나 setFieldValues 메서드는 매개변수가 총 6개로,  
만약 재사용이 필요하다면 재사용하기 힘들 수 있으며,  
시간이 지난 후에 다시 봤을 때 직관적이지 않아 보일 것 같다.  

---

#### **한계**  

---

##### 현재 스타일은 적용할 수 없다.  
먼저 아직까지 큰 필요성을 느끼지 못했으며,  
전체적인 컨트롤은 가능하지만, 세세한 컨트롤을 하기엔 너무 까다로워서 제외하였다.  

---

##### Form 전송으로는 다운로드가 가능하나, Ajax를 통해 다운로드가 불가하다.  
에러는 발생하지 않지만 경고로 아래와 같은 메시지가 발생한다.  
`The provided value 'stream' is not a valid enum value of type XMLHttpRequestResponseType.`  
아마도 Response Content-Type으로 application/octet-stream을 사용하는데  
Ajax가 정상적으로 파싱하지 못한 것으로 보인다. (확실하지 않으니 신뢰하지 말것)   

아직까진 정확한 원인을 찾지 못해 데이터는 Form으로 넘겨야한다.

---

#### **참고자료**

[아파치 POI 공식 홈페이지](https://poi.apache.org/components/spreadsheet/how-to.html#user_api)  

[백기선님의 더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation)  


---