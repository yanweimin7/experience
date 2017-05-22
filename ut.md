单测
PowerMockito mockStatic 不支持mock 父类的static 方法
比如
class B{
public static void test();
}
class A extends B{
}
PowerMockito.mockStatic(A.class)
B.test方法没有被mock，必须同时PowerMockito.mockStatic(B.class)
