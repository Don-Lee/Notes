����������е�ʱ�䣬����������һ��Android�����о���������һЩ���⣬�󲿷ֶ��ǻ��������ռ��ģ��ܶ�ԭ���ߵ�����û���ˣ��ǳ���Ǹ�����������ֱ��������ϵ��thanks.

1��Service ��Thread ������
a)��Thread: thread�ǳ���ִ�е���С��Ԫ������CPU�Ļ�����λ��������thread��ִ��һЩ�첽������
b)��Service: service��android��һ�ֻ��ƣ��������е�ʱ�������LocalService,��ô��Ӧ��Service���������̵߳�main�߳��ϡ������RemoteService,��ô��Ӧ��service�������ڶ������̵�main�߳��ϡ�
��Ȼ������������Ϊʲô��Ҫ��Service�أ���ʵ���android��ϵͳ�����йأ���������Thread��˵��Thread�������Ƕ�����Activity�ģ�Ҳ����˵��Activity��finish֮�������û������ֹͣThread����Thread���run����û��ִ����ϵĻ���ThreadҲ��һֱִ��(PS:��activity���л�����̨ʱ���ϵͳ���ţ�ϵͳ����ʱ���������process).�����������һ�����⣬��activity��finish֮���㲻�ڳ��и�Thread�����á�����һ���棬��û�а취�ڲ�ͬ��activity�ж�ͬһ��Thread���п��ơ�
�ٸ����ӣ�������Thread��Ҫ��ͣ�ĸ�һ��ʱ���Ҫ���ӷ�������ĳ��ͬ���Ļ�����Thread��Ҫ��Activityû��start��ʱ��Ҳ�����С����ʱ����startһ��activity��û�취�ڸ�activity�����֮ǰ������Thread����ˣ������Ҫ����������һ��Service,��Service���洴�������в����Ƹ�Thread�����������˸����⣨��Ϊ�κ�Activity�����Կ���ͬһService����ϵͳҲֻ�ᴴ��һ����Ӧ��Serviceʵ������
�������԰�Service�����һ����Ϣ���񣬶���������κ���Context�ĵط�����Context.startService��Context.stopService��Context.bindService��Context.unbindService������������Ҳ������Service��ע��BroadcastReceiver���������ط�ͨ������broadcast������������Ȼ����Щ����Thread�������ġ�
���ݽ��̵����ȼ���Thread�ں�̨����(Activity stop)�����ȼ����ں�̨���е�Service�����ִ��ϵͳ��Դ���ţ�������ɱ��ǰһ�֣���̨���е�serviceһ�㲻�ᱻɱ�������ɱ����ϵͳ����ʱ����������service��
service���������߳��У������Service������ʱ����Ҫ�������̡߳�

2��Handler��ʲô��
Handler��android�������ṩ��������UI��һ�׻��ƣ�Ҳ��һ����Ϣ������ƣ����ǿ��Է�����Ϣ��Ҳ����ͨ������������Ϣ��
a)��ΪʲôҪ��Handler��
android����Ƶ�ʱ��ͷ�װ��һ����Ϣ���������ݡ�������ơ��������ѭ�����Ļ��ƾ�û�취����UI��Ϣ���ͻ��׳��쳣��android.view.ViewRootImpl$CalledFromWrongThreadException:
Only the original thread that created a view hierarchy can touch its views.��
b)��androidΪʲôҪ���ͨ��Handler���Ƹ���UI�أ�
������������ǽ�����̲߳��������⡣
���������һ��activity�У��ж���߳�ȥ����UI�����Ҷ�û�м������ƣ���ô�����ʲô�������⣿  ���½������
����Ը��µ�UI���������м�������Ļ��ֻ����ʲô�������⣿    1������������UI�ؼ���ø��Ӻ͵�Ч��2���������������ĳЩ���̵�ִ��
���ڶ���������Ŀ��ǣ�android�������ṩ��һ�׸���UI�Ļ��ƣ�����ֻ��Ҫ��ѭ�����Ļ��ƾͿ����ˣ��������ù��Ķ��̵߳����⣬���и���UI�������������̵߳���Ϣ���е���ȥ��������ġ�
c)��handler��ԭ��
Handler
��װ����Ϣ�ķ��ͣ�������̲߳������������⣬��ַ����Messagetarget��Ĭ��������͸��Լ���
Looper
1�����ڲ�����һ����Ϣ����Ҳ����MessageQueue�����е�handler���͵���Ϣ�����������Ϣ����
2����Looper.Looper����������һ����ѭ�������ϵĴ�MessageQueue��ȡ��Ϣ��������Ϣ�ʹ���û����Ϣ������
�ܽ᣺handler��������Ϣ��Looper�������Handler���͵���Ϣ����ֱ�ӷ���Ϣ�ش���Handler�Լ���MessageQueue����һ����Ϣ�洢����
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/handler.png)




3����α�֤Service����ɱ��
a)����߽��̵����ȼ������ͽ��̱�ɱ���ĸ���
    1)������Activity����Ȩ��
    ��������ֻ����������¼�������Ļ����ʱ����1�����ص�Activity�����û�������Activity���١�
    ���ó�������������Ҫ���������Ӧ�ü�ϵͳ�������ڼ�⵽�����¼���һ��ʱ�䣨һ��Ϊ5�������ڣ��ڻ�ɱ����̨���̣��Ѵﵽʡ���Ŀ������
    2)������Notification����Ȩ��
    Android�е�Service�����ȼ�Ϊ4��ͨ��setForeground�ӿڿ��Խ���̨Service����Ϊǰ̨Service���ǽ��̵����ȼ���4����Ϊ2���Ӷ�ʹ���̵����ȼ����������û���ǰ���ڽ����Ľ��̣���ɼ����̵����ȼ�һ�¡�
    3)��AndroidManifest����
    ��AndroidManifest.xml�ļ��ж���intent-filter����ͨ��android:priority="1000"�����������������ȼ���1000�����ֵ������ԽС�����ȼ�Խ�ͣ�ͬʱҲ�����ڹ㲥��
b)���������󣬽�������
    1)������ϵͳ�㲥����
    �����ڷ����ض�ϵͳ�¼�ʱ���磺��Ļ������������������仯�������ȣ���ϵͳ�ᷢ����Ӧ�Ĺ㲥��ͨ����AndroidManifest�С���̬��ע���Ӧ�Ĺ㲥�������������ڷ�����Ӧ���¼����
    2)������ϵͳService��������
    ������Service����ΪSTART_STICKY������ϵͳ������ Service �ҵ����Զ�����
    3��������Native���̽�������
    �������� Linux �е� fork ���ƴ��� Native ���̣��� Native �����м�������̵Ĵ��������̹ҵ����� Native �����������������̽������
    4)���Զ���㲥����
    ����service +broadcast ��ʽ�����ǵ�service��ondestory��ʱ�򣬷���һ���Զ���Ĺ㲥�����յ��㲥��ʱ����������service
c)��ͨ����������
    �����ն˲�ͬ����С���ֻ������� MIUI������С�����͡���Ϊ�ֻ����뻪Ϊ���ͣ������ֻ����Կ��ǽ�����Ѷ�Ÿ�򼫹�������С�������� A/B Test�������Ӧ�ÿɿ��ǽ��� Google �� GCM��
�ο����ͣ�https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653577617&idx=1&sn=623256a2ff94641036a6c9eea17baab8&scene=0#wechat_redirect

4��Activity��Fragment����������
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/activity_fragment.png)




5��Activity������ջ
��AndroidManifest.xml��ʹ��android:launchMode="standard|singleInstance|singleTask|SingleTop"������activity������ջ��

����ջ��һ�ֺ���ȳ��Ľṹ��λ��ջ����Activity���ڽ���״̬��������back��ʱ��ջ�ڵ�activity��һ��һ���ĳ�ջ��������onDestory()���������ջ��û��activity����ôϵͳ�ͻ�������ջ��ÿ��appĬ��ֻ��һ��ջ����app�İ���������.
  standard:��׼ģʽ��ÿ������activity���ᴴ��һ���µĵ�        activityʵ�������ҽ���ѹ������ջջ������������        ��activity�Ƿ��Ѿ����ڡ�
  singleTop:������ģʽ��ջ������ģʽ��������ģʽ��            standardģʽ�������ƣ�����һ�㲻ͬ����Ҫ������        ��Ŀ��activity�Ѿ�λ��ջ��ʱ��ϵͳ�������´���        Ŀ��activity��ʵ��������ֱ�Ӹ������е�activity        ʵ����
  singleTask���ڵ���ģʽ(ջ�ڸ���ģʽ)���������ּ���ģ       ʽ��Activity��ͬһ������ջ��ֻ��һ��ʵ��������       �ô�ģʽ����activityʱ���ɷ�Ϊ�������������
	a�������Ҫ������activity�����ڣ�ϵͳ��ᴴ��activity��ʵ�����������ӵ�ջ����
	b)�����Ҫ������activity�Ѿ�λ��ջ������ʱ��singleTop��ͬ
	c)�����Ҫ������activity�Ѿ����ڣ�����û��λ��ջ����ϵͳ�����λ�ڸ�activity��������е�activity�Ƴ�ջ���Ӷ�ʹ��Ŀ��acitivityת��ջ����
  singleInstance��ȫ�ֵ���ģʽ�����ּ���ģʽ��ϵͳ��֤       ���۴��Ǹ�����ջ������Ŀ��activity��ֻ�ᴴ��һ       ��Ŀ��activityʵ��������ʹ��һ��ȫ�µĵ�����ջ       ��װ�ظ�activityʵ����
	�������ַ�ʽ����activity����Ϊ����2�������
	a)�������Ҫ������Ŀ��activity�����ڣ�ϵͳ���ȴ���һ��ȫ�µ�ջ���ٴ���Ŀ��activity��ʵ���������������µ�ջ��ջ����
	b)�����Ҫ������activity�Ѿ����ڣ�������λ���ĸ�Ӧ�ó����У�����λ���Ǹ�ջ�У�ϵͳ����Ѹ�activity���ڵ�ջת��ǰ̨���Ӷ�ʹ��activity��ʾ������

