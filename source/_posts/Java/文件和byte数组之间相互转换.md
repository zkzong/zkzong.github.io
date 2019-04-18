## 文件转换成byte数组

文件转换成byte数组有两种方式：

### 1. 传统方式
```
File file = new File("/temp/abc.txt");
//init array with file length
byte[] bytesArray = new byte[(int) file.length()];

FileInputStream fis = new FileInputStream(file);
fis.read(bytesArray); //read file into bytes[]
fis.close();

return bytesArray;
```
### 2. NIO方式
```
String filePath = "/temp/abc.txt";

byte[] bFile = Files.readAllBytes(new File(filePath).toPath());
//or this
byte[] bFile = Files.readAllBytes(Paths.get(filePath));
```

### 完整实例

下面的例子展示了如何把读取的文件内容转换成byte数组，使用了传统的`FileInputStream`和`java.nio`两种方式。
```
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * @author zkzong
 * @date 2017/10/20
 */
public class FileToArrayOfBytes {
    public static void main(String[] args) {

        try {

            // convert file to byte[]
            byte[] bFile = readBytesFromFile("test.txt");

            //java nio
            //byte[] bFile = Files.readAllBytes(new File("test.txt").toPath());
            //byte[] bFile = Files.readAllBytes(Paths.get("test.txt"));

            // save byte[] into a file
            Path path = Paths.get("test2.txt");
            Files.write(path, bFile);

            System.out.println("Done");

            //Print bytes[]
            for (int i = 0; i < bFile.length; i++) {
                System.out.print((char) bFile[i]);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    private static byte[] readBytesFromFile(String filePath) {

        FileInputStream fileInputStream = null;
        byte[] bytesArray = null;

        try {

            File file = new File(filePath);
            bytesArray = new byte[(int) file.length()];

            //read file into bytes[]
            fileInputStream = new FileInputStream(file);
            fileInputStream.read(bytesArray);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }

        return bytesArray;

    }
}
```

## byte数组转换成文件

byte数组转换成文件也有两种方式：

### 1. 传统方式
```
FileOutputStream fos = new FileOutputStream(fileDest);
fos.write(bytesArray);
fos.close();
```
### 2. NIO方式
```
Path path = Paths.get(fileDest);
Files.write(path, bytesArray);
```

### 完整实例

下面的例子展示了如何把读取的文件内容转换成byte数组，并把保存的byte数组转换成一个新的文件，使用了传统的`try-catch`、JDK 7的`try-resources`和`java.nio`两种方式。
```
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * @author zkzong
 * @date 2017/10/20
 */
public class ArrayOfBytesToFile {
    private static final String UPLOAD_FOLDER = "/";

    public static void main(String[] args) {

        FileInputStream fileInputStream = null;

        try {

            File file = new File("test.txt");
            byte[] bFile = new byte[(int) file.length()];

            //read file into bytes[]
            fileInputStream = new FileInputStream(file);
            fileInputStream.read(bFile);

            //save bytes[] into a file
            writeBytesToFile(bFile, UPLOAD_FOLDER + "test1.txt");
            writeBytesToFileClassic(bFile, UPLOAD_FOLDER + "test2.txt");
            writeBytesToFileNio(bFile, UPLOAD_FOLDER + "test3.txt");

            System.out.println("Done");

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    }

    //Classic, < JDK7
    private static void writeBytesToFileClassic(byte[] bFile, String fileDest) {

        FileOutputStream fileOuputStream = null;

        try {
            fileOuputStream = new FileOutputStream(fileDest);
            fileOuputStream.write(bFile);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileOuputStream != null) {
                try {
                    fileOuputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    //Since JDK 7 - try resources
    private static void writeBytesToFile(byte[] bFile, String fileDest) {

        try (FileOutputStream fileOuputStream = new FileOutputStream(fileDest)) {
            fileOuputStream.write(bFile);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    //Since JDK 7, NIO
    private static void writeBytesToFileNio(byte[] bFile, String fileDest) {

        try {
            Path path = Paths.get(fileDest);
            Files.write(path, bFile);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

}
```
