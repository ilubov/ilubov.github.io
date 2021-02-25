---
title: EasyExcel导出合并表头的Excel
date: 2021-02-01 22:11:19
tags:
    - java
categories:
    - java
---

### EasyExcel导出合并表头的Excel
[文档地址](https://alibaba-easyexcel.github.io/index.html)

#### maven
```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.2.7</version>
</dependency>
```

#### 工具类
```
import cn.hutool.core.date.DatePattern;
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.CharsetUtil;
import cn.hutool.core.util.URLUtil;
import cn.hutool.http.ContentType;
import cn.hutool.http.Header;
import com.alibaba.excel.EasyExcelFactory;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.support.ExcelTypeEnum;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.google.common.collect.Maps;
import org.apache.poi.util.StringUtil;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
import java.util.Map;

public class EasyExcelUtil {

    public static void export(HttpServletResponse response, List<List<String>> headList,
                              List<List<Object>> dataList, String filename) throws IOException {
        String fileName = new String((filename + DateUtil.format(DateUtil.date(), DatePattern.PURE_DATE_PATTERN))
                .getBytes(), CharsetUtil.CHARSET_UTF_8);
        ExcelTypeEnum excelType = ExcelTypeEnum.XLSX;
        ServletOutputStream out = response.getOutputStream();
        response.setContentType(ContentType.MULTIPART.getValue());
        response.setCharacterEncoding(CharsetUtil.UTF_8);
        response.setHeader(Header.CONTENT_DISPOSITION.getValue(),
                "attachment;filename=" + URLUtil.encode(fileName, StringUtil.UTF8) + excelType.getValue());
        ExcelWriter writer = EasyExcelFactory
                .write(out)
                .excelType(excelType)
                .head(headList)
                .build();
        Map<Integer, Integer> columnWidth = Maps.newHashMap();
        for (int i = 0; i < headList.size(); i++) {
            List<String> list = headList.get(i);
            columnWidth.put(i, list.get(list.size() - 1).length() * 1000);
        }
        WriteSheet sheet = new WriteSheet();
        sheet.setColumnWidthMap(columnWidth);
        sheet.setSheetName(filename);
        writer.write(dataList, sheet);
        writer.finish();
        out.close();
    }
}
```

#### 测试接口
```
import com.cloudyoung.competing.utils.EasyExcelUtils;
import com.google.common.collect.ImmutableList;
import com.google.common.collect.Lists;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@RequestMapping("/test")
@RestController
public class TestController {

    @GetMapping("/export")
    public void export(HttpServletResponse response) throws IOException {
        List<List<String>> headList = Lists.newArrayList();
        List<List<Object>> dataList = Lists.newArrayList();
        String filename = "test";
        for (int j = 1; j < 5; j++) {
            List<Object> data = Lists.newArrayList();
            for (int i = 1; i < 10; i++) {
                if (j == 1) {
                    String h1, h2 = String.format("head%s", i);
                    if (i > 5) {
                        h1 = String.format("head%s", 2);
                    } else {
                        h1 = String.format("head%s", 1);
                    }
                    headList.add(ImmutableList.of(h1, h2));
                }
                data.add(i);
            }
            dataList.add(data);
        }
        EasyExcelUtils.export(response, headList, dataList, filename);
    }
}
```
