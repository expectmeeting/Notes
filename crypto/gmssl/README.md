# 国密算法

## SM2算法

sm2公钥格式用ASN.1描述，SM2PublicKey::=BIT STRING

- 非压缩公钥

sm2的公钥是sm2椭圆曲线上的一个点Q(x,y)，每个分类长度为256位，非压缩公钥的第一个字节为0x04，SM2PublicKey = （0x04||x||y），因此非压缩公钥长65字节，私钥为32字节。

- 压缩公钥

由于公钥中已知x分量和y分量的奇偶性即可推导出y分量，因此压缩公钥为SM2PublicKey = （0x02||x）或（0x03||x），0x02表示y分量为偶数，0x03表示y分量为奇数。

## gmssl命令行命令



~~~shell
# 生成私钥：
gmssl ecparam -genkey -name sm2p256v1 -text -out user.key
gmssl sm2 -genkey -out sm2.pem -text
# 生成公钥
gmssl pkey -pubout -in user.key -out pub.key
~~~

## 重要函数

~~~c
 eckey = PEM_read_bio_EC_PUBKEY(in, NULL, NULL, NULL);//从文件类型的BIO中读取EC_KEY，公钥格式为PEM
str = EC_POINT_point2hex(NULL, eckey->pub_key,
                                 POINT_CONVERSION_UNCOMPRESSED, NULL);
~~~



## 重要结构体

~~~c
struct ec_key_st {
    const EC_KEY_METHOD *meth;
    ENGINE *engine;
    int version;
    EC_GROUP *group;
    EC_POINT *pub_key;
    BIGNUM *priv_key;
    unsigned int enc_flag;
    point_conversion_form_t conv_form;
    int references;
    int flags;
    CRYPTO_EX_DATA ex_data;
    CRYPTO_RWLOCK *lock;
};

struct ec_point_st {
    const EC_METHOD *meth;
    /*
     * All members except 'meth' are handled by the method functions, even if
     * they appear generic
     */
    BIGNUM *X;
    BIGNUM *Y;
    BIGNUM *Z;                  /* Jacobian projective coordinates: * (X, Y,
                                 * Z) represents (X/Z^2, Y/Z^3) if Z != 0 */
    int Z_is_one;               /* enable optimized point arithmetics for
                                 * special case */
};


char *EC_POINT_point2hex(const EC_GROUP *group, const EC_POINT *p,
                                 point_conversion_form_t form, BN_CTX *ctx);

一些术语说明：

1)    椭圆曲线的阶(order of a curve)

椭圆曲线所有点的个数，包含无穷远点；

2)    椭圆曲线上点的阶(order of a point)

P为椭圆曲线上的点，nP=无穷远点，n取最小整数，既是P的阶；

3)    基点(base point)

椭圆曲线参数之一，用G表示，是椭圆曲线上都一个点；

4)    余因子(cofactor)

椭圆曲线的余因子，用h表示，为椭圆曲线点的个数/基点的阶

5)    椭圆曲线参数：

素数域:

(p，a，b，G，n，h)

其中，p为素数，确定Fp，a和b确定椭圆曲线方程，G为基点，n为G的阶，h为余因子。

二进制扩展域:

(m，f(x)，a，b，G，n，h)

其中，m确定F2m，f(x)为不可约多项式，a和b用于确定椭圆曲线方程，G为基点，n为G的阶，h为余因子。

6)    椭圆曲线公钥和私钥

       椭圆曲线的私钥是一个随机整数，小于n；

       椭圆曲线的公钥是椭圆曲线上的一个点：Q＝私钥*G。

20.2  openssl的ECC实现
       Openssl实现了ECC算法。ECC算法系列包括三部分：ECC算法(crypto/ec)、椭圆曲线数字签名算法ECDSA (crypto/ecdsa)以及椭圆曲线密钥交换算法ECDH(crypto/ecdh)。

       研究椭圆曲线需要注意的有：

1)      数据结构

?  椭圆曲线数据结构：EC_GROUP，该结构不仅包含各个参数，还包含了各种算法；

?  点的表示：EC_POINT，其中的大数X、Y和Z为雅克比投影坐标；

?  EC_CURVE_DATA：用于内置椭圆曲线，包含了椭圆曲线的各个参数；

?  密钥结构

椭圆曲线密钥数据结构如下，定义在crypto/ec_lcl.h中，对用户是透明的。

    struct ec_key_st
 
{
 
                     int                 version;
 
                     EC_GROUP   *group;
 
                     EC_POINT    *pub_key;
 
                     BIGNUM       *priv_key;
 
                     /* 其他项 */
 
}
 

2)   密钥生成

对照公钥和私钥的表示方法，非对称算法不同有各自的密钥生成过程。椭圆曲线的密钥生成实现在crytpo/ec/ec_key.c中。Openssl中，内置的椭圆曲线密钥生成时，首先用户需要选取一种椭圆曲线(openssl的crypto/ec_curve.c中内置实现了67种，调用EC_get_builtin_curves获取该列表)，然后根据选择的椭圆曲线计算密钥生成参数group，最后根据密钥参数group来生公私钥。

3）签名值数据结构

非对称算法不同，签名的结果表示也不一样。与DSA签名值一样，ECDSA的签名结果表示为两项。ECDSA的签名结果数据结构定义在crypto/ecdsa/ecdsa.h中，如下：

       typedef struct ECDSA_SIG_st

{

              BIGNUM *r;

              BIGNUM *s;

} ECDSA_SIG;

4)    签名与验签

对照签名结果，研究其是如何生成的。crypto/ecdsa/ ecs_sign.c实现了签名算法，crypto/ecdsa/ ecs_vrf.c实现了验签。

       5）  密钥交换

              研究其密钥交换是如何进行的；crypto/ecdh/ech_ossl.c实现了密钥交换算法。

 

文件说明：

EC_METHOD实现

ec2_smpl.c            F2m  二进制扩展域上的EC_METHOD实现；

ecp_mont.c     Fp   素数域上的EC_METHOD实现，(Montgomery 蒙哥马利)

ecp_smpl.c     Fp   素数域上的EC_METHOD实现；

ecp_nist.c       Fp   素数域上的EC_METHOD实现；

ec2_mult.c

       F2m上的乘法；

ec2_smpt.c

       F2m上的压缩算法；

ec_asn1.c

       asn1编解码；

ec_check.c

       椭圆曲线检测；

ec_curve.c

       内置的椭圆曲线，

       NID_X9_62_prime_field：X9.62的素数域；

       NID_X9_62_characteristic_two_field：X9.62的二进制扩展域；

       NIST：美国国家标准

ec_cvt.c

       给定参数生成素数域和二进制扩展域上的椭圆曲线；

ec_err.c

       错误处理；

ec_key.c

       椭圆曲线密钥EC_KEY函数；

ec_lib.c

       通用库实现，一般会调用底层的EC_METHOD方法；

ec_mult.c

       This file implements the wNAF-based interleaving multi-exponentation method乘法；

ec_print.c

数据与椭圆曲线上点的相互转化；

ectest.c

       测试源码，可以参考此源码学习椭圆曲线函数。

ec.h

       对外头文件；

ec_lcl.h

                            内部头文件，数据结构一般在此定义。

20.3  主要函数
20.3.1参数设置
1)    int EC_POINT_set_affine_coordinates_GF2m(const EC_GROUP *group, EC_POINT *point,

        const BIGNUM *x, const BIGNUM *y, BN_CTX *ctx)

说明：设置二进制域椭圆曲线上点point的几何坐标；

2)    int EC_POINT_set_affine_coordinates_GFp(const EC_GROUP *group, EC_POINT *point,

        const BIGNUM *x, const BIGNUM *y, BN_CTX *ctx)

说明：设置素数域椭圆曲线上点point的几何坐标；

3)   int EC_POINT_set_compressed_coordinates_GF2m(const EC_GROUP *group, EC_POINT *point,const BIGNUM *x, int y_bit, BN_CTX *ctx)

说明：二进制域椭圆曲线，给定压缩坐标x和y_bit参数，设置point的几何坐标；用于将Octet-String转化为椭圆曲线上的点；

4)    int EC_POINT_set_compressed_coordinates_GFp(const EC_GROUP *group, EC_POINT *point,        const BIGNUM *x, int y_bit, BN_CTX *ctx)

说明：素数域椭圆曲线，给定压缩坐标x和y_bit参数，设置point的几何坐标；用于将Octet-String转化为椭圆曲线上的点；

5)    int EC_POINT_set_Jprojective_coordinates_GFp(const EC_GROUP *group, EC_POINT *point,

        const BIGNUM *x, const BIGNUM *y, const BIGNUM *z, BN_CTX *ctx)

说明：素数域椭圆曲线group，设置点point的投影坐标系坐标x、y和z；

6)    int EC_POINT_set_to_infinity(const EC_GROUP *group, EC_POINT *point)

说明：将点point设为无穷远点

7)    int EC_GROUP_set_curve_GF2m(EC_GROUP *group, const BIGNUM *p, const BIGNUM *a, const BIGNUM *b, BN_CTX *ctx)

说明：设置二进制域椭圆曲线参数；

8)    int EC_GROUP_set_curve_GFp(EC_GROUP *group, const BIGNUM *p, const BIGNUM *a, const BIGNUM *b, BN_CTX *ctx)

说明：设置素数域椭圆曲线参数；

9)    int EC_GROUP_set_generator(EC_GROUP *group, const EC_POINT *generator, const BIGNUM *order, const BIGNUM *cofactor)

说明：设置椭圆曲线的基G；generator、order和cofactor为输入参数；

10)  size_t EC_GROUP_set_seed(EC_GROUP *group, const unsigned char *p, size_t len)

说明：设置椭圆曲线随机数，用于生成a和b；

11)  EC_GROUP *EC_GROUP_new_curve_GF2m(const BIGNUM *p, const BIGNUM *a, const BIGNUM *b, BN_CTX *ctx)

说明：生成二进制域上的椭圆曲线，输入参数为p，a和b；

12)  EC_GROUP *EC_GROUP_new_curve_GFp(const BIGNUM *p, const BIGNUM *a, const BIGNUM *b, BN_CTX *ctx)

说明：生成素数域上的椭圆曲线。

20.3.2参数获取
1)    const EC_POINT *EC_GROUP_get0_generator(const EC_GROUP *group)

说明：获取椭圆曲线的基(G)；

2)    unsigned char *EC_GROUP_get0_seed(const EC_GROUP *group)

说明：获取椭圆曲线参数的随机数，该随机数可选，用于生成椭圆曲线参数中的a和b；

3）  int EC_GROUP_get_basis_type(const EC_GROUP *group)

说明：获取二进制域多项式的类型；

4）  int EC_GROUP_get_cofactor(const EC_GROUP *group, BIGNUM *cofactor, BN_CTX *ctx)

说明：获取椭圆曲线的余因子。cofactor为X9.62中定义的h，值为椭圆曲线点的个数/基点的阶，即：cofactor = #E(Fq)/n。

5）  int EC_GROUP_get_curve_GF2m(const EC_GROUP *group, BIGNUM *p, BIGNUM *a, BIGNUM *b, BN_CTX *ctx)

说明：获取二元域椭圆曲线的三个参数，其中p可表示多项式；

6)    int EC_GROUP_get_curve_GFp(const EC_GROUP *group, BIGNUM *p, BIGNUM *a, BIGNUM *b, BN_CTX *ctx)

说明：获取素数域椭圆曲线的三个参数；

7)    int EC_GROUP_get_curve_name(const EC_GROUP *group)

说明：获取椭圆曲线名称，返回其NID；

8)    int EC_GROUP_get_degree(const EC_GROUP *group)

说明：获取椭圆曲线密钥长度。对于素数域Fp来说，是大数p的长度；对二进制域F2m来说，等于m；

9)    int EC_GROUP_get_order(const EC_GROUP *group, BIGNUM *order, BN_CTX *ctx)

说明：获取椭圆曲线的阶；

10)  int EC_GROUP_get_pentanomial_basis(const EC_GROUP *group, unsigned int *k1,

       unsigned int *k2, unsigned int *k3)

int EC_GROUP_get_trinomial_basis(const EC_GROUP *group, unsigned int *k)

说明：获取多项式参数；

11)   int EC_POINT_get_affine_coordinates_GF2m(const EC_GROUP *group, const EC_POINT *point,BIGNUM *x, BIGNUM *y, BN_CTX *ctx)

说明：获取二进制域椭圆曲线上某个点的x和y的几何坐标；

12)  int EC_POINT_get_affine_coordinates_GFp(const EC_GROUP *group, const EC_POINT *point,        BIGNUM *x, BIGNUM *y, BN_CTX *ctx)

说明：获取素数域上椭圆曲线上某个点的x和y的几何坐标；

13)  int EC_POINT_get_Jprojective_coordinates_GFp(const EC_GROUP *group, const EC_POINT *point,BIGNUM *x, BIGNUM *y, BIGNUM *z, BN_CTX *ctx)

说明：获取素数域椭圆曲线上某个点的x、y和z的投影坐标系坐标。

20.3.3转化函数
1)    EC_POINT *EC_POINT_bn2point(const EC_GROUP *group,                           const BIGNUM *bn,EC_POINT *point,BN_CTX *ctx)

说明：将大数转化为椭圆曲线上的点；

2)    EC_POINT *EC_POINT_hex2point(const EC_GROUP *group,                            const char *buf,EC_POINT *point,BN_CTX *ctx)

说明：将buf中表示的十六进制数据转化为椭圆曲线上的点；

3)    int BN_GF2m_poly2arr(const BIGNUM *a, unsigned int p[], int max)

说明：将大数转化为多项式的各个项；

4)    int BN_GF2m_arr2poly(const unsigned int p[], BIGNUM *a)

说明：将多项式的各个项转化为大数；

5)    int EC_POINT_make_affine(const EC_GROUP *group, EC_POINT *point, BN_CTX *ctx)

说明：将椭圆曲线group上点的point转化为几何坐标系；

6)    int EC_POINT_oct2point(const EC_GROUP *group, EC_POINT *point,

        const unsigned char *buf, size_t len, BN_CTX *ctx)

说明：将buf中点数据转化为椭圆曲线上的点，len为数据长度；

7)    BIGNUM *EC_POINT_point2bn(const EC_GROUP *group,const EC_POINT *point,                          point_conversion_form_t form,BIGNUM *ret,BN_CTX *ctx)

说明：将椭圆曲线上的点转化为大数，其中from为压缩方式，可以是POINT_CONVERSION_COMPRESSED、POINT_CONVERSION_UNCOMPRESSED或POINT_CONVERSION_HYBRID，可参考x9.62；

8)    char *EC_POINT_point2hex(const EC_GROUP *group, const EC_POINT *point,                        point_conversion_form_t form,BN_CTX *ctx)

说明：将椭圆曲线上的点转化为十六进制，并返回该结果；

9)    size_t EC_POINT_point2oct(const EC_GROUP *group, const EC_POINT *point, point_conversion_form_t form,unsigned char *buf, size_t len, BN_CTX *ctx)

说明：将椭圆曲线上的点转化为Octet-String，可分两次调用，用法见EC_POINT_point2bn的实现。

20.3.4其他函数
1)    size_t EC_get_builtin_curves(EC_builtin_curve *r, size_t nitems)

文件：ec_curve.c

说明：获取内置的椭圆曲线。当输入参数r为NULL或者nitems为0时，返回内置椭圆曲线的个数，否则将各个椭圆曲线信息存放在r中。

示例：
#include <openssl/ec.h>
 
int   main()
 
{
 
       EC_builtin_curve   *curves = NULL;
 
    size_t                           crv_len = 0, n = 0;
 
       int                               nid,ret;
 
       EC_GROUP                 *group = NULL;
 
 
 
    crv_len = EC_get_builtin_curves(NULL, 0);
 
    curves = OPENSSL_malloc(sizeof(EC_builtin_curve) * crv_len);
 
       EC_get_builtin_curves(curves, crv_len);
 
       for (n=0;n<crv_len;n++)
 
       {
 
              nid = curves[n].nid;
 
               group=NULL;
 
              group = EC_GROUP_new_by_curve_name(nid);
 
              ret=EC_GROUP_check(group,NULL);
 
       }
 
       OPENSSL_free(curves);
 
       return 0;
 
}
 

2)    const EC_METHOD *EC_GF2m_simple_method(void)

说明：返回二进制域上的方法集EC_METHOD

3）  const EC_METHOD *EC_GFp_mont_method(void)

const EC_METHOD *EC_GFp_nist_method(void)

const EC_METHOD *EC_GFp_simple_method(void)

返回素数域上的方法集EC_METHOD

4）  int EC_GROUP_check(const EC_GROUP *group, BN_CTX *ctx)

说明：检查椭圆曲线，成功返回1。

5）  int EC_GROUP_check_discriminant(const EC_GROUP *group, BN_CTX *ctx)

说明：检查椭圆曲线表达式。对于素数域的椭圆曲线来说，该函数会调用ec_GFp_simple_group_check_discriminant函数，主要检查4*a^3 + 27*b^2 != 0 (mod p)。而对于二进制域的椭圆曲线，会调用ec_GF2m_simple_group_check_discriminant, 检查y^2 + x*y = x^3 + a*x^2 + b 是否是一个椭圆曲线并且 b !=0。

6）  int EC_GROUP_cmp(const EC_GROUP *a, const EC_GROUP *b, BN_CTX *ctx)

说明：通过比较各个参数来确定两个椭圆曲线是否相等；

7）  int EC_GROUP_copy(EC_GROUP *dest, const EC_GROUP *src)

EC_GROUP *EC_GROUP_dup(const EC_GROUP *a)

说明：椭圆曲线拷贝函数；

9）  EC_GROUP *EC_GROUP_new_by_curve_name(int nid)

说明：根据NID获取内置的椭圆曲线；

10)  int EC_KEY_check_key(const EC_KEY *eckey)

说明：检查椭圆曲线密钥；

11)  int EC_KEY_generate_key(EC_KEY *eckey)

说明：生成椭圆曲线公私钥；

12)  int EC_KEY_print(BIO *bp, const EC_KEY *x, int off)

说明：将椭圆曲线密钥信息输出到bio中，off为缩进量；

13)  int EC_POINT_add(const EC_GROUP *group, EC_POINT *r, const EC_POINT *a, const EC_POINT *b, BN_CTX *ctx)

说明：椭圆曲线上点的加法；

14)  int EC_POINT_invert(const EC_GROUP *group, EC_POINT *a, BN_CTX *ctx)

说明：求椭圆曲线上某点a的逆元，a既是输入参数，也是输出参数；

15)  int EC_POINT_is_at_infinity(const EC_GROUP *group, const EC_POINT *point)

说明：判断椭圆曲线上的点point是否是无穷远点；

16)  int EC_POINT_is_on_curve(const EC_GROUP *group, const EC_POINT *point, BN_CTX *ctx)

说明：判断一个点point是否在椭圆曲线上；

17)  int     ECDSA_size

       说明：获取ECC密钥大小字节数。

18）ECDSA_sign

       说明：签名，返回1表示成功。

19）ECDSA_verify

       说明：验签，返回1表示合法。

20）EC_KEY_get0_public_key

       说明：获取公钥。

21）EC_KEY_get0_private_key

       说明：获取私钥。

22）ECDH_compute_key

       说明：生成共享密钥

23）EC_KEY *d2i_ECPrivateKey(EC_KEY **a, const unsigned char **in, long len)

       说明：DER解码将椭圆曲线密钥；

24）int     i2d_ECPrivateKey(EC_KEY *a, unsigned char **out)

       说明：将椭圆曲线密钥DER编码；

20.4  编程示例
下面的例子生成两对ECC密钥，并用它做签名和验签，并生成共享密钥。

#include <string.h>
 
#include <stdio.h>
 
#include <openssl/ec.h>
 
#include <openssl/ecdsa.h>
 
#include <openssl/objects.h>
 
#include <openssl/err.h>
 
 
 
int   main()
 
{
 
       EC_KEY              *key1,*key2;
 
       EC_POINT            *pubkey1,*pubkey2;
 
       EC_GROUP          *group1,*group2;
 
       int                        ret,nid,size,i,sig_len;
 
       unsigned char  *signature,digest[20];
 
       BIO                     *berr;
 
       EC_builtin_curve   *curves;
 
       int                               crv_len;
 
       char              shareKey1[128],shareKey2[128];
 
       int                        len1,len2;
 
 
 
       /* 构造EC_KEY数据结构 */
 
       key1=EC_KEY_new();
 
       if(key1==NULL)
 
       {
 
              printf("EC_KEY_new err!\n");
 
              return -1;
 
       }
 
       key2=EC_KEY_new();
 
       if(key2==NULL)
 
       {
 
              printf("EC_KEY_new err!\n");
 
              return -1;
 
       }
 
       /* 获取实现的椭圆曲线个数 */
 
       crv_len = EC_get_builtin_curves(NULL, 0);
 
       curves = (EC_builtin_curve *)malloc(sizeof(EC_builtin_curve) * crv_len);
 
       /* 获取椭圆曲线列表 */
 
       EC_get_builtin_curves(curves, crv_len);
 
       /*
 
       nid=curves[0].nid;会有错误，原因是密钥太短
 
       */
 
       /* 选取一种椭圆曲线 */
 
       nid=curves[25].nid;
 
       /* 根据选择的椭圆曲线生成密钥参数group */
 
       group1=EC_GROUP_new_by_curve_name(nid);
 
       if(group1==NULL)
 
       {
 
              printf("EC_GROUP_new_by_curve_name err!\n");
 
              return -1;
 
       }
 
       group2=EC_GROUP_new_by_curve_name(nid);
 
       if(group1==NULL)
 
       {
 
              printf("EC_GROUP_new_by_curve_name err!\n");
 
              return -1;
 
       }
 
       /* 设置密钥参数 */
 
       ret=EC_KEY_set_group(key1,group1);
 
       if(ret!=1)
 
       {
 
              printf("EC_KEY_set_group err.\n");
 
              return -1;
 
       }
 
       ret=EC_KEY_set_group(key2,group2);
 
       if(ret!=1)
 
       {
 
              printf("EC_KEY_set_group err.\n");
 
              return -1;
 
       }
 
       /* 生成密钥 */
 
       ret=EC_KEY_generate_key(key1);
 
       if(ret!=1)
 
       {
 
              printf("EC_KEY_generate_key err.\n");
 
              return -1;
 
       }
 
       ret=EC_KEY_generate_key(key2);
 
       if(ret!=1)
 
       {
 
              printf("EC_KEY_generate_key err.\n");
 
              return -1;
 
       }
 
       /* 检查密钥 */
 
       ret=EC_KEY_check_key(key1);
 
       if(ret!=1)
 
       {
 
              printf("check key err.\n");
 
              return -1;
 
       }
 
       /* 获取密钥大小 */
 
       size=ECDSA_size(key1);
 
       printf("size %d \n",size);
 
       for(i=0;i<20;i++)
 
              memset(&digest[i],i+1,1);
 
       signature=malloc(size);
 
       ERR_load_crypto_strings();
 
       berr=BIO_new(BIO_s_file());
 
       BIO_set_fp(berr,stdout,BIO_NOCLOSE);
 
       /* 签名数据，本例未做摘要，可将digest中的数据看作是sha1摘要结果 */
 
       ret=ECDSA_sign(0,digest,20,signature,&sig_len,key1);
 
       if(ret!=1)
 
       {
 
              ERR_print_errors(berr);
 
              printf("sign err!\n");
 
              return -1;
 
       }
 
       /* 验证签名 */
 
       ret=ECDSA_verify(0,digest,20,signature,sig_len,key1);
 
       if(ret!=1)
 
       {
 
              ERR_print_errors(berr);
 
              printf("ECDSA_verify err!\n");
 
              return -1;
 
       }
 
       /* 获取对方公钥，不能直接引用 */
 
       pubkey2 = EC_KEY_get0_public_key(key2);
 
       /* 生成一方的共享密钥 */
 
       len1=ECDH_compute_key(shareKey1, 128, pubkey2, key1, NULL);
 
       pubkey1 = EC_KEY_get0_public_key(key1);
 
       /* 生成另一方共享密钥 */
 
       len2=ECDH_compute_key(shareKey2, 128, pubkey1, key2, NULL);
 
       if(len1!=len2)
 
       {
 
              printf("err\n");
 
       }
 
       else
 
       {
 
              ret=memcmp(shareKey1,shareKey2,len1);
 
              if(ret==0)
 
                     printf("生成共享密钥成功\n");
 
              else
 
                     printf("生成共享密钥失败\n");
 
       }
 
       printf("test ok!\n");
 
       BIO_free(berr);
 
       EC_KEY_free(key1);
 
       EC_KEY_free(key2);
 
       free(signature);
 
       free(curves);
 
       return 0;
 
}
 

更多底层函数的使用示例可参考ec/ectest.c，特别是用户自行定义椭圆曲线参数。
~~~