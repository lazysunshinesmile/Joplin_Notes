Java常用api_

Java常用api:

Java常用api:
public byte[] Bitmap2Bytes(Bitmap bitmap) {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();// outputstream

[    bitmap.compress(Bitmap.CompressFormat](http://bitmap.compress(bitmap.compressformat/).*PNG*, 100, baos);

    return baos.toByteArray();// 转为byte数组
    // return Base64.encodeToString(appicon, Base64.DEFAULT).toCharArray();
}

public Bitmap Bytes2Bimap(byte[] b) {
    if (b.length != 0) {
        return BitmapFactory.*decodeByteArray*(b, 0, b.length);
    } else {
        return null;
    }
}