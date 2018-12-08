---
title: 验证码和二维码生成

date: 2018-05-05 07:07:00

updated: 2018-05-05 07:07:00

tags:
- 工具类

categories: Java

permalink: captcha-qrcode
---



## 验证码

涉及到Session操作，所以在控制层中生成

~~~java
@GetMapping("verify_code")
@SneakyThrows
public RequestResult verifyCode() {
	try (ByteArrayOutputStream jpegOutputStream = new ByteArrayOutputStream()) {
            String createText = defaultKaptcha.createText();
        	// 向HttpSession中存入验证码字符串
            RequestContextUtil.session().setAttribute(VERIFY_CODE, createText);
            BufferedImage challenge = defaultKaptcha.createImage(createText);
            ImageIO.write(challenge, "jpg", jpegOutputStream);
            // 生成到本地监听路径
            String location = beginningMindProperties.getFile().getLocation();
            String fullFileName = System.currentTimeMillis() + ".jpg";
            FileUtils.writeByteArrayToFile(new File(location + File.separator + fullFileName),
                    jpegOutputStream.toByteArray());
            // 拼接出映射到验证码图片的URL，并返回给前端
            String fullMapping = beginningMindProperties.getAddress() + beginningMindProperties.getFile().getMapping();
            return RequestResult.success(fullMapping + "/" + fullFileName);
        } catch (IOException e) {
            throw new ServiceException(e);
        }
}
~~~



## 二维码

~~~xml
<!--zxing-->
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.3.2</version>
</dependency>
~~~

 

~~~java
/**
 * 工具类：二维码
 *
 * @author Deolin
 */
@UtilityClass
@Log4j2
public class QRCodeUtil {

    /**
     * 生成二维码(QRCode)图片
     *
     * @param content 二维码信息
     * @param imgPath 文件路径
     */
    public static void generate(String content, String imgPath) {
        try {
            log.info(content);
            log.info(imgPath);
            QRCodeUtil.encode(content, null, imgPath, true);
        } catch (Exception e) {
            throw new RuntimeException("生成二维码失败", e);
        }
    }

    private static final String FORMAT = "PNG";

    private static final int QRCODE_SIZE = 300;

    private static final int LOGO_WIDTH = 140;

    private static final int LOGO_HEIGHT = 140;

    private static BufferedImage createImage(String content, String logoPath, boolean needCompress) throws Exception {
        Hashtable<EncodeHintType, Object> hints = new Hashtable<>();
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H);
        hints.put(EncodeHintType.CHARACTER_SET, StandardCharsets.UTF_8);
        hints.put(EncodeHintType.MARGIN, 1);
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, QRCODE_SIZE, QRCODE_SIZE,
                hints);
        int width = bitMatrix.getWidth();
        int height = bitMatrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
                image.setRGB(x, y, bitMatrix.get(x, y) ? 0xFF000000 : 0xFFFFFFFF);
            }
        }
        if (logoPath == null || "".equals(logoPath)) {
            return image;
        }
        // 插入图片
        QRCodeUtil.insertImage(image, logoPath, needCompress);
        return image;
    }

    /**
     * 插入LOGO
     *
     * @param source 二维码图片
     * @param logoPath LOGO图片地址
     * @param needCompress 是否压缩
     */
    private static void insertImage(BufferedImage source, String logoPath, boolean needCompress) throws Exception {
        File file = new File(logoPath);
        if (!file.exists()) {
            throw new Exception("logo file not found.");
        }
        Image src = ImageIO.read(new File(logoPath));
        int width = src.getWidth(null);
        int height = src.getHeight(null);
        if (needCompress) { // 压缩LOGO
            if (width > LOGO_WIDTH) {
                width = LOGO_WIDTH;
            }
            if (height > LOGO_HEIGHT) {
                height = LOGO_HEIGHT;
            }
            Image image = src.getScaledInstance(width, height, Image.SCALE_SMOOTH);
            BufferedImage tag = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            Graphics g = tag.getGraphics();
            g.drawImage(image, 0, 0, null); // 绘制缩小后的图
            g.dispose();
            src = image;
        }
        // 插入LOGO
        Graphics2D graph = source.createGraphics();
        int x = (QRCODE_SIZE - width) / 2;
        int y = (QRCODE_SIZE - height) / 2;
        graph.drawImage(src, x, y, width, height, null);
        Shape shape = new RoundRectangle2D.Float(x, y, width, width, 6, 6);
        graph.setStroke(new BasicStroke(3f));
        graph.draw(shape);
        graph.dispose();
    }

    /**
     * 生成二维码(内嵌LOGO)
     * 二维码文件名随机，文件名可能会有重复
     *
     * @param content 内容
     * @param logoPath LOGO地址
     * @param destPath 存放目录
     * @param needCompress 是否压缩LOGO
     */
    public static String encode(String content, String logoPath, String destPath,
            boolean needCompress) throws Exception {
        BufferedImage image = QRCodeUtil.createImage(content, logoPath, needCompress);
        File destFile = new File(destPath);
        ImageIO.write(image, FORMAT, destFile);
        return destFile.getName();
    }

    /**
     * 生成二维码(内嵌LOGO)
     * 调用者指定二维码文件名
     *
     * @param content 内容
     * @param logoPath LOGO地址
     * @param destPath 存放目录
     * @param fileName 二维码文件名
     * @param needCompress 是否压缩LOGO
     */
    public static String encode(String content, String logoPath, String destPath, String fileName,
            boolean needCompress) throws Exception {
        BufferedImage image = QRCodeUtil.createImage(content, logoPath, needCompress);
        mkdirs(destPath);
        fileName = fileName.substring(0, fileName.indexOf(".") > 0 ? fileName.indexOf(".") : fileName.length())
                + "." + FORMAT.toLowerCase();
        ImageIO.write(image, FORMAT, new File(destPath + "/" + fileName));
        return fileName;
    }

    /**
     * 当文件夹不存在时，mkdirs会自动创建多层目录，区别于mkdir．
     * (mkdir如果父目录不存在则会抛出异常)
     *
     * @param destPath 存放目录
     */
    private static void mkdirs(String destPath) {
        File file = new File(destPath);
        if (!file.exists() && !file.isDirectory()) {
            file.mkdirs();
        }
    }

    /**
     * 生成二维码(内嵌LOGO)
     *
     * @param content 内容
     * @param logoPath LOGO地址
     * @param destPath 存储地址
     */
    public static String encode(String content, String logoPath, String destPath) throws Exception {
        return QRCodeUtil.encode(content, logoPath, destPath, false);
    }

    /**
     * 生成二维码
     *
     * @param content 内容
     * @param destPath 存储地址
     * @param needCompress 是否压缩LOGO
     */
    public static String encode(String content, String destPath, boolean needCompress) throws Exception {
        return QRCodeUtil.encode(content, null, destPath, needCompress);
    }

    /**
     * 生成二维码
     *
     * @param content 内容
     * @param destPath 存储地址
     */
    public static String encode(String content, String destPath) throws Exception {
        return QRCodeUtil.encode(content, null, destPath, false);
    }

    /**
     * 生成二维码(内嵌LOGO)
     *
     * @param content 内容
     * @param logoPath LOGO地址
     * @param output 输出流
     * @param needCompress 是否压缩LOGO
     */
    public static void encode(String content, String logoPath, OutputStream output, boolean needCompress)
            throws Exception {
        BufferedImage image = QRCodeUtil.createImage(content, logoPath, needCompress);
        ImageIO.write(image, FORMAT, output);
    }

    /**
     * 生成二维码
     *
     * @param content 内容
     * @param output 输出流
     */
    public static void encode(String content, OutputStream output) throws Exception {
        QRCodeUtil.encode(content, null, output, false);
    }

    /**
     * 解析二维码
     *
     * @param file 二维码图片
     */
    public static String decode(File file) throws Exception {
        BufferedImage image;
        image = ImageIO.read(file);
        if (image == null) {
            return null;
        }
        BufferedImageLuminanceSource source = new BufferedImageLuminanceSource(image);
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        Hashtable<DecodeHintType, Object> hints = new Hashtable<>();
        hints.put(DecodeHintType.CHARACTER_SET, StandardCharsets.UTF_8);
        Result result = new MultiFormatReader().decode(bitmap, hints);
        String resultStr = result.getText();
        return resultStr;
    }

    /**
     * 解析二维码
     *
     * @param path 二维码图片地址
     */
    public static String decode(String path) throws Exception {
        return QRCodeUtil.decode(new File(path));
    }

}
~~~

