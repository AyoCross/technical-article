
����pycharm��һ��õ�python�������ߣ�����ʮ��ǿ�󣬲��Ҷ�ƽ̨֧�֣�Windows/MacOS/Linux�����ٷ��ṩ������Դ�汾��[pycharm Community��Ѱ汾���ص�ַ](https://www.jetbrains.com/pycharm/download/ "pycharm���ص�ַ")��


# pycharm���ճ������г��ù��ܼ�飺

1. �ϵ���ԣ�������Visual Studio��ʵʱ�鿴����/�ڴ�ʹ�������֧��django/flask�ȹ��̵ĵ���:
![django����ͼ](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E8%B0%83%E8%AF%95%E5%9B%BE.png "")
2. �Զ�����������ǶPEP8��׼���������¼����������ͳһ�����ڹ������÷�����

  >(1) pip install autopep8��
  
  >(2) ѡ��˵���File���C>��Settings���C>��Tools���C>��External Tools���C>����Ӻ���ӹ���
  ![��װautopep8](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%AE%89%E8%A3%85autopep8.png "")
  
  >(3)��д������
  ```
  Name��Autopep8 (��������д)
  Tools settings:      
  Programs��autopep8
  Parameters��--in-place --aggressive --ignore=E123,E133,E50 $FilePath$
  Working directory��$ProjectFileDir$
  ```
  ![autopep8������](https://raw.githubusercontent.com/AyoCross/usual_pics/master/autopep8%E9%85%8D%E7%BD%AE%E9%A1%B9.png "")

3. ��Ƕgit���ܣ�������а汾���ơ�
4. ֧��ģ��ģ�壬���ڹ�����ͳһע�ͷ�񣬱������ߵȡ�
5. �Զ�����ģ��ȫ�ֱ���������λ�ã�λ���л�ʮ�ַ��㡣
![�ļ�structure](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E6%96%87%E4%BB%B6structure.png "")
6. ���뵥Ԫ���Թ��ܣ�����ݵý��е�Ԫ���ԣ���ߴ���³���ԡ�

����Ϊ���������г��ÿ�ݼ�������
1. Ctrl+Click(��Ctrl+B)���ɲ鿴Դ�룬�˽����ԭ���鿴���������ѡ�к������ƣ�Ctrl+Click���ɲ鿴���øú�����λ�ã����ɷ��ء�
2. �Զ����룬Alt+Enter���Զ���ʾ������ģ������������ѡ��Settings->general->autoimport->python :show popup��![�Զ�����](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E8%87%AA%E5%8A%A8%E5%BC%95%E5%85%A5.png "")
3. ���볣�ô���Ctrl+J�������ظ��Ͷ���
![���ô���](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%B8%B8%E7%94%A8%E4%BB%A3%E7%A0%81.png "")

4. 4.	���ٲ����ļ�Ctrl+E��������ʵ��ļ���Ctrl+Shift+E������༭�����ļ����������ڴ��͹����У���ͳ�İ�������ļ��ǳ���Ч�����Ctrl+Tab�л�ǰһ���򿪵��ļ����л�Ч������������
![���ٲ����ļ�](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%BF%AB%E9%80%9F%E6%9F%A5%E6%89%BE%E6%96%87%E4%BB%B6.png "")
10. ����������˫��Shift�����������ļ�����������������������������Ŀ¼��������Ŀ¼�ļ��������ڹؼ���ǰ���б��/��
11. ��ʷճ���壬Ctrl+Shift+V�ɷ�����ʷճ���塣
12. ������ʾ���룬Settings-Keymap�н����Զ��塣
13. ���⻻�У� Shfit+Enter��
14. 9.	�Ż����룬Ctrl+Alt+O��ȥ��û��ʹ�õ����룬�淶����˳��
15. ע��ѡ���У�Ctrl+/
16. 

�������⣬pycharm֧�ֶ�python���棬֧��virtualenv�������á���汾��ͬʱ�������ӷ��㡣

�����������棬pycharm�����Զ������壬�﷨������ɫ��һϵ�����������ã�����ʹд������ӳ������֡�

---
# ��¼1��Win/Mac��ݼ���ԭͼ������ϱ�̣�
![pycharmWindows�¿ͻ��˵Ŀ�ݼ�](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEWin.png "")

[Win��ݼ�����ͼ����](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEWin.png "")

![pycharm Mac�¿ͻ��˵Ŀ�ݼ�](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEMac.jpg "")

[Mac��ݼ�����ͼ����](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEMac.jpg "")

---
# ��¼2��PEP8��׼
[PEP 8��׼-Ӣ��ԭ��](https://www.python.org/dev/peps/pep-0008/ "PEP 8��׼-Ӣ��ԭ��")

## PEP8��׼����ժҪ

### ���벼��

> 1.�����̶�Ϊ4���ո񣬷�ֹ��ͬƽ̨֮���table������һ�£�

> 2.ÿ�еĳ������Ϊ79���ֽڣ�

> 3��import����ʱ����׼����ǰ�������ǵ������⣬�Լ��Զ�����ļ����룻

### ע��

> 	ģ�鶥��Ҫ��ģ��ע�ͣ����Լ��Զ��庯��Ҫ���书�ܣ��Լ���γ���ע�ͣ�����������⴦����ĩ������ע�ͣ�����# FIXME/TODO�ȸ���ע�ͣ�

### �������

> 1.����ʹ�ÿ��ļ����������׻����ĳ�������

> 2.�������쳣��������Ҫ������ĸ��д�Ĺ����ڲ��࣬Ҫ����ǰ���»��ߣ�

> 3.���������������Сд��Ϊ�����ӿɶ��Կ������»��߷ָ���

> 4.�����뷽���Ĳ���������self��Ϊʵ�������ĵ�һ��������cls��Ϊ�෽���ĵ�һ��������

> 5.��������д��Ϊ�����ӿɶ��Կ������»��߷ָ���

### ������ƽ���

> 1.������None�������ַ��Ƚ�ʱ��Ҫʹ��is or is not����Ҫ�õ��ڲ�����

> 2.������һ���쳣��ʱ��Ҫ����ϸ�쳣��������except Exception��һ���յ�except:��佫�Ჶ�� SystemExit �� KeyboardInterrrupt �쳣�����ʹ�ú�����Ctrl+C���ж�һ�����򣬲��һ����������������⣩��

> 3.try�еĴ��빦��Խ��ȷԽ�ã���Ҫ��һ��try�������������ݣ�

> 4.��isinstance(object, type)�������ͱȽϣ�


