### hex to ip
```
using System;
using System.Globalization;

public static class IpConverter
{
    // 예외 방식: 잘못된 입력이면 ArgumentException 발생
    public static string Hex8ToIp(string hex)
    {
        if (!TryHex8ToIp(hex, out string ip))
            throw new ArgumentException("유효한 8자리 hex 문자열이 아닙니다. 예: \"C0A80101\" 또는 \"0xC0A80101\"");

        return ip;
    }

    // 안전 방식: 성공하면 true와 ip 반환, 실패하면 false
    public static bool TryHex8ToIp(string hex, out string ip)
    {
        ip = null;
        if (hex == null) return false;

        hex = hex.Trim();
        if (hex.StartsWith("0x", StringComparison.OrdinalIgnoreCase))
            hex = hex.Substring(2);

        if (hex.Length != 8) return false;

        // 2자리씩 잘라서 0-255 범위로 파싱
        var parts = new string[4];
        for (int i = 0; i < 4; i++)
        {
            string byteHex = hex.Substring(i * 2, 2);
            if (!byte.TryParse(byteHex, NumberStyles.HexNumber, CultureInfo.InvariantCulture, out byte b))
                return false;
            parts[i] = b.ToString();
        }

        ip = string.Join(".", parts);
        return true;
    }
}

```
