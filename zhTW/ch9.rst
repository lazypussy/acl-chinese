.. highlight:: cl
   :linenothreshold: 0

Chapter 9 數字 (Numbers)
***************************************************

處理數字是 Common Lisp 的強項之一。Common Lisp 有著豐富的數值型別 (numeric types)，而 Common Lisp 操作數字的特性與任何語言比起來更受人喜愛。

9.1 型別 (Types)
==================================

Common Lisp 提供了四種不同種類的數字：整數 (integers)、浮點數 (floating-point numbers)、比值 (ratios) 與複數 (complex number)。本章所描述的函數適用於所有種類的數字。有幾個不能用在複數的會特別註明。

一個整數是寫成一串數字： ``2001`` 。一個浮點數是可以寫成一串包含小數點的數字， ``253.72`` ，或是用科學表示法， ``2.5372e2`` 。一個比值是寫成一個由整數組成的分數： ``2/3`` 。而複數 ``a+bi`` 是寫成 ``#c(a b)`` ，其中 ``a`` 與 ``b`` 是任兩個同樣種類的實數 (real number)。

判斷式 ``integerp`` , ``floatp`` 以及 ``complexp`` 對於相對應的數字種類回傳真。圖 9.1 展示了數值型別的層級 (hierarchy of numeric types)。

.. figure:: https://github.com/acl-translation/acl-chinese/raw/master/images/Figure-9.1.png

**圖 9.1: 數值型別**

以下是某些通用的經驗法則，來決定一個計算過程會回傳何種數字：

1. 如果一個數值函數 (numeric function)接受一個或多個浮點數作為參數，則回傳值會是一個浮點數 (或是一個由浮點數組成的複數)。所以 ``(+ 1.0 2)`` 求值成 ``3.0`` ，而 ``(+ #c(0 1.0) 2)`` 求值成 ``#c(2.0 1.0)`` 。

2. 可約分的比值會被轉換成整數。所以 ``(/ 10 2)`` 會回傳 ``5`` 。

3. 若計算過程中複數的虛部會變成 ``0`` ，則複數會被轉成實數 。所以 ``(+ #c(1 -1) #c(2 1))`` 求值成 ``3`` 。

第二、第三個規則可以在參數被讀取時直接應用，所以：

::

	> (list (ratiop 2/2) (complexp #c(1 0)))
	(NIL NIL)

9.2 轉換及取出 (Conversion and Extraction)
==============================================================

Lisp 提供函數來做四種不同類型的數字的轉換 (converting)及取出位數 (extracting component)。函數 ``float`` 將任何實數轉換成一個浮點數:

::

	> (mapcar #'float '(1 2/3 .5))
	(1.0 0.6666667 0.5)

將數字轉成整數未必需要轉換，因為它可能牽涉到某些資訊的喪失。函數 ``truncate`` 回傳任何實數的整數部分:

::

	> (truncate 1.3)
	1
	0.29999995

第二個回傳值是傳入的參數減去第一個回傳值。(會有 0.00000005 的差是因為浮點數的計算本來就不精確。)

函數 ``floor`` 與 ``ceiling`` 以及 ``round`` 也從它們的參數中導出整數。使用 ``floor`` 回傳小於等於其參數的最大整數，而 ``ceiling`` 回傳大於或等於其參數的最小整數，我們可以將 ``mirror?`` (46 頁，譯註: 3.11 節)改成可以找出所有迴文 (palindromes)的版本:

::

	(defun palindrome? (x)
	  (let ((mid (/ (length x) 2)))
	    (equal (subseq x 0 (floor mid))
	           (reverse (subseq x (ceiling mid))))))

和 ``truncate`` 一樣， ``floor`` 與 ``ceiling`` 也返回傳入參數與第一個回傳值的差作為第二個值。

::

	> (floor 1.5)
	1
	0.5

實際上，我們可以把 ``truncate`` 想成是這樣定義的:

::

	(defun our-truncate (n)
	    (if (> n 0)
	        (floor n)
	        (ceiling n)))

函數 ``round`` 回傳最接近其參數的整數。當參數與兩個整數的距離相等時， Common Lisp 和很多程式語言一樣，不會往上取 (round up)整數。而是取最近的偶數:

::

	> (mapcar #'round '(-2.5 -1.5 1.5 2.5))
	(-2 -2 2 2)

在某些數值應用中這是好事，因為捨入誤差 (rounding error)傾向於互相抵消。然而如果用戶期望你的程式將某些值取整數時，你必須自己提供這個功能。 [1]_ 與其他的函數一樣， ``round`` 返回傳入參數與第一個回傳值的差作為第二個值。

函數 ``mod`` 僅回傳 ``floor`` 會回傳的第二個值；而 ``rem`` 回傳 ``truncate`` 會回傳的第二個值。我們在 94 頁 (譯註: 5.7 節)使用了 ``mod`` 來決定一個數是否可被另一個整除，以及 127 頁(譯註: 7.4 節)用來找出環狀緩衝區 (ring buffer)中，元素實際的位置。

關於實數，函數 ``signum`` 回傳 ``1`` , ``0`` 或 ``-1`` ，取決於它的參數是正、零或負數。函數 ``abs`` 回傳其參數的絕對值。因此 ``(* (abs x) (signum x))`` 等於 ``x`` 。

::

	> (mapcar #'signum '(-2 -0.0 0.0 0 .5 3))
	(-1 -0.0 0.0 0 1.0 1)

在某些應用裡， ``-0.0`` 可能自成一格 (in its own right)，如上所示。其實功能上它幾乎沒有差異，因為數值 ``-0.0`` 與 ``0.0`` 有一樣的行為。

比值與複數概念上是兩部分結構。(譯註: 像 **Cons** 這樣的兩部分結構) 函數 ``numerator`` 與 ``denominator`` 回傳一個比值或整數所對應的部份。 (如果數字是整數，前者回傳該數字，而後者回傳 ``1`` 。)函數 ``realpart`` 與 ``imgpart`` 回傳任何數字的實數與虛數部分。 (如果數字不是複數，前者回傳該數字，後者回傳 ``0`` 。)

函數 ``random`` 接受一個整數或浮點數。一個這樣形式的表達式 ``(random n)`` 回傳一個大於或小於等於 ``n`` 的數字，並有著與 ``n`` 相同的型別。

9.3 比較 (Comparison)
================================

判斷式 ``=`` 當其參數數值上相等時 –– 即兩者的差為零時，回傳真。

::

	> (= 1 1.0)
	T
	> (eql 1 1.0)
	NIL

``=`` 比起 ``eql`` 來得寬鬆，但它的參數需要是同樣型別。

用來比較數字的判斷式為 ``<`` (小於), ``<=`` (小於等於), ``=`` (等於), ``>=`` (大於等於), ``>`` (大於) 以及 ``/=`` (不同)。以上所有皆接受一個或多個參數。只有一個參數時，它們全回傳真。

::

	(<= w x y z)

等同於一個二元運算元的結合 (conjunction)，應用至每一對參數上:

::

	(and (<= w x) (<= x y) (<= y z))

由於 ``/=`` 若它的兩個參數不等於時回傳真，表達式

::

	(/= w x y z)

等同於

::

	(and (/= w x) (/= w y) (/= w z)
	     (/= x y) (/= y z) (/= y z))

特殊的判斷式 ``zerop`` , ``plusp`` 與 ``minusp`` 接受一個參數，分別於參數 ``=`` , ``>`` , ``<`` 零時，回傳真。雖然 ``-0.0`` (如果實現有使用它) 前面有個負號，但它 ``=`` 零，

::

	> (list (minusp -0.0) (zerop -0.0))
	(NIL T)

因此使用 ``zerop`` 而不是 ``minusp`` 。

判斷式 ``oddp`` 與 ``evenp`` 只能用在整數。前者只對奇數回傳真，後者只對偶數回傳真。

本節定義的判斷式中，只有 ``=`` , ``/=`` 與 ``zerop`` 可以用在複數。

函數 ``max`` 與 ``min`` 分別回傳其參數的最大值與最小值。兩者至少需要給一個參數:

::

	> (list (max 1 2 3 4 5) (min 1 2 3 4 5))
	(5 1)

如果參數有包含浮點數的話，結果的型別取決於各家實現。

9.4 算術 (Arithematic)
===================================================

用來做加減的函數是 ``+`` 與 ``-`` 。兩者皆可接受任何數量的參數，包括沒有參數，在沒有參數的情況下回傳 ``0`` 。(譯註: ``-`` 在沒有參數的情況下會報錯，至少要一個參數) 一個這樣形式的表達式 ``(- n)`` 回傳 ``-n`` 。一個這樣形式的表達式

::

	(- x y z)

等同於

::

	(- (- x y) z)

有兩個函數 ``1+`` 與 ``1-`` ，分別將參數加上 ``1`` 與減去 ``1`` 並回傳。 ``1-`` 有一點誤導，因為 ``(1- x)`` 回傳 ``x-1`` 而不是 ``1-x`` 。

巨集 ``incf`` 及	 ``decf`` 分別增加與減少參數。一個這樣形式的表達式 ``(incf x n)`` 類似於 ``(setf x (+ x n))`` 的效果，而 ``(decf x n)`` 類似於 ``(setf x (- x n))`` 的效果。這兩個情況裡，第二個參數是選擇性的並預設為 ``1`` 。

用來做乘法的函數是 ``*`` 。接受任何數量的參數。沒有給參數時回傳 ``1`` 。否則回傳參數的乘積。

除法函數 ``/`` 至少預期一個參數。一個這樣形式的呼叫 ``(/ n)`` 等同於 ``(/ 1 n)`` ，

::

	> (/ 3)
	1/3

而一個這樣形式的呼叫

::

	(/ x y z)

等同於

::

	(/ (/ x y) z)

注意 ``-`` 與 ``/`` 兩者在這方面的相似性。

當給定兩個整數時， ``/`` 若第一個不是第二個的倍數時，會回傳一個比值:

::

	> (/ 365 12)
	365/12

舉例來說，如果你試著找出平均每一個月有多長，你可能會有頂層在逗你玩的想法。在這個情況下，你需要的是對比值呼叫 ``float`` ，而不是對兩個整數做 ``/`` 。

::

	> (float 365/12)
	30.416666

9.5 指數 (Exponentiation)
=======================================

要找到 :math:`x^n` 我們呼叫 ``(expt x n)`` ，

::

	> (expt 2 5)
	32

而要找到 :math:`log_nx` 我們呼叫 ``(log x n)`` :

::

	> (log 32 2)
	5.0

通常回傳一個浮點數。

要找到 :math:`e^x` 有一個特別的函數 ``exp`` ，

::

	> (exp 2)
	7.389056

而要找到一個自然對數，你可以使用 ``log`` 就好，因為第二個參數預設為 ``e`` :

::

	> (log 7.389056)
	2.0

要找到立方根，你可以呼叫 ``expt`` 用一個比值作為第二個參數，

::

	> (expt 27 1/3)
	3.0

但要找到平方根，函數 ``sqrt`` 會比較快:

::

	> (sqrt 4)
	2.0

9.6 三角函數 (Trigometric Functions)
=======================================

常數 ``pi`` 是 ``π`` 的浮點表示法。它的精度 (precision)取決於各家實現。函數 ``sin`` , ``cos`` 及 ``tan`` 分別可以找到正弦、餘弦及正交函數，其中角度以徑度 (radian)表示:

::

	> (let ((x (/ pi 4)))
	    (list (sin x) (cos x) (tan x)))
	(0.7071067811865475d0 0.7071067811865476d0 1.0d0)
	;;; 譯註: CCL 1.8  SBCL 1.0.55 下的結果是
	;;; (0.7071067811865475D0 0.7071067811865476D0 0.9999999999999999D0)

這些函數全部接受負數及複數參數。

函數 ``asin`` , ``acos`` 及 ``atan`` 實現了正弦、餘弦及正交的反函數 (inverse)。參數介於 ``-1`` 與 ``1`` 之間（包含）時， ``asin`` 與 ``acos`` 回傳實數。

雙曲 (hyperbolic)正弦、餘弦及正交分別由 ``sinh`` , ``cosh`` 及 ``tanh`` 實現。它們的反函數同樣為 ``asinh`` , ``acosh`` 以及 ``atanh`` 。

9.7 表示法 (Representations)
=======================================

Common Lisp 對於整數的大小沒有限制。可以塞進一個字 (word)記憶體的小整數稱為定數 (fixnums)。當一個計算過程整數無法塞入一個字 (word)時，Lisp 切換至使用多個記憶體字的表示法（一個大數 「bignum」)。所以一個整數的大小限制取決於實體記憶體，而不是語言。

常數 ``most-positive-fixnum`` 與 ``most-negative-fixnum`` 表示了一個實現不使用大數 (bignum)可表示的最大數字幅度 (magnitude)。在很多實現裡，它們為：

::

	> (values most-positive-fixnum most-negative-fixnum)
	536870911
	-536870912
	;;; 譯註: CCL 1.8 的結果為
	1152921504606846975
	-1152921504606846976
	;;; SBCL 1.0.55 的結果為
	4611686018427387903
	-4611686018427387904

判斷式 ``typep`` 接受一個參數及一個型別名稱，並回傳指定型別的參數。所以，

::

	> (typep 1 'fixnum)
	T
	> (type (1+ most-positive-fixnum) 'bignum)
	T

浮點數字的數值限制是取決於各家實現的。 Common Lisp 提供了最多四種型別的浮點數：短浮點 ``short-float`` 、 單浮點 ``single-float`` 、雙浮點 ``double-float`` 以及長浮點 ``long-float`` 。Common Lisp 的實現不需要用不同的格式來表示這四種型別（很少實現這麼做）。

一般來說，短浮點應可塞入一個字 (word)，單浮點與雙浮點提供普遍的單與雙精度浮點數的概念，而長浮點，如果想要的話可以是很大的數。但一個實現可以使這四個型別沒有區別，也是完全沒有問題的。

你可以指定你想要何種格式的浮點數，當一個數字是用科學表示法時，可以通過將 ``e`` 替換為 ``s`` ``f`` ``d`` ``l`` 來得到不同的浮點數。(你也可以使用大寫，這對長浮點來說是個好主意，因為 ``l`` 看起來太像 ``1`` 了。)所以要表示最大的 ``1.0`` 你可以寫 ``1L0`` 。

(譯註: ``s`` 為短浮點、 ``f`` 為單浮點、 ``d`` 為雙浮點、 ``l`` 為長浮點。)

在給定的實現裡，用十六個全域常數標明了每個格式的限制。它們的名字是這種形式: ``m-s-f`` ，其中 ``m`` 是 ``most`` 或 ``least`` ， ``s`` 是 ``positive`` 或 ``negative`` ，而 ``f`` 是四種浮點數之一。

浮點數乾涸與溢出被 Common Lisp 視為錯誤 :

(譯註: 這裡調皮了一下，使用了乾涸。我們說一個 stack 滿了要 push 時叫做溢出 (overflow)，stack 為空又要 pop 時叫做下溢「underflow」，但是下溢聽起來以為是褲子濕了…)

::

	> (* most-positive-long-float 10)
	Error: floating-point-overflow

9.8 範例：追蹤光線 (Example: Ray-Tracing)
==========================================

作為一個數值應用的範例，本節示範了如何撰寫一個光線追蹤器 (ray-tracer)。光線追蹤是一個高級的 (deluxe)渲染算法: 它產生出逼真的圖像，但需要花點時間。

要產生一個 3D 的圖像，我們至少需要定義四件事: 一個觀測點 (eye)、一個或多個光源、一個由一個或多個平面所組成的模擬世界 (simulated world)，以及一個作為通往這個世界的窗戶的平面 (圖像平面「image plane」)。我們產生出的是模擬世界投影在圖像平面區域的圖像。

讓光線追蹤如此不尋常的是，我們如何找到這個投影: 我們一個一個像素地沿著圖像平面走，追蹤回到模擬世界裡的光線。這個方法帶來三個主要的優勢: 它讓我們容易得到現實世界的光學效應 (optical effect)，如透明度 (transparency)、反射光 (reflected light)以及產生陰影 (cast shadows)；它讓我們可以直接用任何我們想要的幾何的物體，來定義出模擬的世界，而不需要用多變形 (polygons)來建構它們；以及它很簡單實現。

::

	(defun sq (x) (* x x))

	(defun mag (x y z)
	  (sqrt (+ (sq x) (sq y) (sq z))))

	(defun unit-vector (x y z)
	  (let ((d (mag x y z)))
	    (values (/ x d) (/ y d) (/ z d))))

	(defstruct (point (:conc-name nil))
	  x y z)

	(defun distance (p1 p2)
	  (mag (- (x p1) (x p2))
	       (- (y p1) (y p2))
	       (- (z p1) (z p2))))

	(defun minroot (a b c)
	  (if (zerop a)
	      (/ (- c) b)
	      (let ((disc (- (sq b) (* 4 a c))))
	        (unless (minusp disc)
	          (let ((discrt (sqrt disc)))
	            (min (/ (+ (- b) discrt) (* 2 a))
	                 (/ (- (- b) discrt) (* 2 a))))))))

**圖 9.2 實用數學函數**

圖 9.2 包含了我們在光線追蹤器裡會需要用到的一些實用數學函數。第一個 ``sq`` ，回傳其參數的平方。下一個 ``mag`` ，回傳一個給定 ``x`` ``y`` ``z`` 所組成向量的大小 (magnitude)。這個函數被接下來兩個函數用到。我們在 ``unit-vector`` 用到了，此函數回傳三個數值，來表示與單位向量有著同樣方向的向量，其中向量是由 ``x`` ``y`` ``z`` 所組成的:

::

	> (multiple-value-call #'mag (unit-vector 23 12 47))
	1.0

我們在 ``distance`` 也用到了 ``mag`` ，它回傳三維空間中，兩點的距離。（定義 ``point`` 結構來有一個 ``nil`` 的 ``conc-name`` 意味著欄位存取的函數會有跟欄位一樣的名字: 舉例來說， ``x`` 而不是 ``point-x`` 。)

最後 ``minroot`` 接受三個實數， ``a`` , ``b`` 與 ``c`` ，並回傳滿足等式 :math:`ax^2+bx+c=0` 的最小實數 ``x`` 。當 ``a`` 不為 0 時，這個等式的根由下面這個熟悉的式子給出:

.. math::

	x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a}

圖 9.3 包含了定義一個最小光線追蹤器的程式碼。 它產生通過單一光源照射的黑白圖像，與觀測點 (eye)處於同個位置。 (結果看起來像是閃光攝影術 (flash photography)拍出來的)

``surface`` 結構會用來表示模擬世界中的物體。更精確的說，它會被 ``included`` 至定義具體種類物體的結構裡，像是球體 (spheres)。 ``surface`` 結構本身只包含一個欄位: 一個 ``color`` 範圍從 0 (黑色) 至 1 (白色)。

::

	(defstruct surface  color)

	(defparameter *world* nil)
	(defconstant eye (make-point :x 0 :y 0 :z 200))

	(defun tracer (pathname &optional (res 1))
	  (with-open-file (p pathname :direction :output)
	    (format p "P2 ~A ~A 255" (* res 100) (* res 100))
	    (let ((inc (/ res)))
	      (do ((y -50 (+ y inc)))
	          ((< (- 50 y) inc))
	        (do ((x -50 (+ x inc)))
	            ((< (- 50 x) inc))
	          (print (color-at x y) p))))))

	(defun color-at (x y)
	  (multiple-value-bind (xr yr zr)
	                       (unit-vector (- x (x eye))
	                                    (- y (y eye))
	                                    (- 0 (z eye)))
	    (round (* (sendray eye xr yr zr) 255))))

	(defun sendray (pt xr yr zr)
	  (multiple-value-bind (s int) (first-hit pt xr yr zr)
	    (if s
	        (* (lambert s int xr yr zr) (surface-color s))
	        0)))

	(defun first-hit (pt xr yr zr)
	  (let (surface hit dist)
	    (dolist (s *world*)
	      (let ((h (intersect s pt xr yr zr)))
	        (when h
	          (let ((d (distance h pt)))
	            (when (or (null dist) (< d dist))
	              (setf surface s hit h dist d))))))
	    (values surface hit)))

	(defun lambert (s int xr yr zr)
	  (multiple-value-bind (xn yn zn) (normal s int)
	    (max 0 (+ (* xr xn) (* yr yn) (* zr zn)))))

**圖 9.3 光線追蹤。**

圖像平面會是由 x 軸與 y 軸所定義的平面。觀測者 (eye) 會在 z 軸，距離原點 200 個單位。所以要在圖像平面可以被看到，插入至 ``*worlds*`` 的表面 (一開始為 ``nil``)會有著負的 z 座標。圖 9.4 說明了一個光線穿過圖像平面上的一點，並擊中一個球體。

.. figure:: https://github.com/acl-translation/acl-chinese/raw/master/images/Figure-9.4.png

**圖 9.4: 追蹤光線。**

函數 ``tracer`` 接受一個路徑名稱，並寫入一張圖片至對應的檔案。圖片檔案會用一種簡單的 ASCII 稱作 PGM 的格式寫入。默認情況下，圖像會是 100x100 。我們 PGM 檔案的標頭 (headers) 會由標籤 ``P2`` 組成，伴隨著指定圖片寬度 (breadth)與高度 (height)的整數，初始為 100，單位為 pixel，以及可能的最大值 (255)。檔案剩餘的部份會由 10000 個介於 0 (黑)與 1 (白)整數組成，代表著 100 條 100 像素的水平線。

圖片的解析度可以通過給入明確的 ``res`` 來調整。舉例來說，如果 ``res`` 是 ``2`` ，則同樣的圖像會被渲染成 200x200 。

圖片是一個在圖像平面 100x100 的正方形。每一個像素代表著穿過圖像平面抵達觀測點的光的數量。要找到每個像素光的數量， ``tracer`` 呼叫 ``color-at`` 。這個函數找到從觀測點至該點的向量，並呼叫 ``sendray`` 來追蹤這個向量回到模擬世界的軌跡； ``sandray`` 會回傳一個數值介於 0 與 1 之間的亮度 (intensity)，之後會縮放成一個 0 至 255 的整數來顯示。

要決定一個光線的亮度， ``sendray`` 需要找到光是從哪個物體所反射的。要辦到這件事，我們呼叫 ``first-hit`` ，此函數研究在 ``*world*`` 裡的所有平面，並回傳光線最先抵達的平面（如果有的話）。如果光沒有擊中任何東西， ``sendray`` 僅回傳背景顏色，按慣例是 ``0`` (黑色)。如果光線有擊中某物的話，我們需要找出在光擊中時，有多少數量的光照在該平面。

`朗伯定律 <http://zh.wikipedia.org/zh-tw/%E6%AF%94%E5%B0%94%EF%BC%8D%E6%9C%97%E4%BC%AF%E5%AE%9A%E5%BE%8B>`_ 告訴我們，由平面上一點所反射的光的強度，正比於該點的單位法向量 (unit normal vector) *N* (這裡是與平面垂直且長度為一的向量)與該點至光源的單位向量 *L* 的點積 (dot-product):

.. math::

	i = N·L

如果光剛好照到這點， *N* 與 *L* 會重合 (coincident)，則點積會是最大值， ``1`` 。如果將在這時候將平面朝光轉 90 度，則 *N* 與 *L* 會垂直，則兩者點積會是 ``0`` 。如果光在平面後面，則點積會是負數。

在我們的程式裡，我們假設光源在觀測點 (eye)，所以 ``lambert`` 使用了這個規則來找到平面上某點的亮度 (illumination)，回傳我們追蹤的光的單位向量與法向量的點積。

在 ``sendray`` 這個值會乘上平面的顏色 (即便是有好的照明，一個暗的平面還是暗的)來決定該點之後總體亮度。

為了簡單起見，我們在模擬世界裡會只有一種物體，球體。圖 9.5 包含了與球體有關的程式碼。球體結構包含了 ``surface`` ，所以一個球體會有一種顏色以及 ``center`` 和 ``radius`` 。呼叫 ``defsphere`` 添加一個新球體至世界裡。

::

	(defstruct (sphere (:include surface))
	  radius center)

	(defun defsphere (x y z r c)
	  (let ((s (make-sphere
	             :radius r
	             :center (make-point :x x :y y :z z)
	             :color  c)))
	    (push s *world*)
	    s))

	(defun intersect (s pt xr yr zr)
	  (funcall (typecase s (sphere #'sphere-intersect))
	           s pt xr yr zr))

	(defun sphere-intersect (s pt xr yr zr)
	  (let* ((c (sphere-center s))
	         (n (minroot (+ (sq xr) (sq yr) (sq zr))
	                     (* 2 (+ (* (- (x pt) (x c)) xr)
	                             (* (- (y pt) (y c)) yr)
	                             (* (- (z pt) (z c)) zr)))
	                     (+ (sq (- (x pt) (x c)))
	                        (sq (- (y pt) (y c)))
	                        (sq (- (z pt) (z c)))
	                        (- (sq (sphere-radius s)))))))
	    (if n
	        (make-point :x  (+ (x pt) (* n xr))
	                    :y  (+ (y pt) (* n yr))
	                    :z  (+ (z pt) (* n zr))))))

	(defun normal (s pt)
	  (funcall (typecase s (sphere #'sphere-normal))
	           s pt))

	(defun sphere-normal (s pt)
	  (let ((c (sphere-center s)))
	    (unit-vector (- (x c) (x pt))
	                 (- (y c) (y pt))
	                 (- (z c) (z pt)))))

**圖 9.5 球體。**

函數 ``intersect`` 判斷與何種平面有關，並呼叫對應的函數。在此時只有一種， ``sphere-intersect`` ，但 ``intersect`` 是寫成可以容易擴展處理別種物體。

我們要怎麼找到一束光與一個球體的交點 (intersection)呢？光線是表示成點 :math:`p =〈x_0,y_0,x_0〉` 以及單位向量 :math:`v =〈x_r,y_r,x_r〉` 。每個在光上的點可以表示為 :math:`p+nv` ，對於某個 *n* –– 即 :math:`〈x_0+nx_r,y_0+ny_r,z_0+nz_r〉` 。光擊中球體的點的距離至中心 :math:`〈x_c,y_c,z_c〉` 會等於球體的半徑 *r* 。所以在下列這個交點的方程式會成立:

.. math::

	r = \sqrt{ (x_0 + nx_r + x_c)^2 + (y_0 + ny_r + y_c)^2 + (z_0 + nz_r + z_c)^2 }

這會給出

.. math::

	an^2 + bn + c = 0

其中

.. math::

	a = x_r^2 + y_r^2 + z_r^2\\b = 2((x_0-x_c)x_r + (y_0-y_c)y_r + (z_0-z_c)z_r)\\c = (x_0-x_c)^2 + (y_0-y_c)^2 + (z_0-z_c)^2 - r^2

要找到交點我們只需要找到這個二次方程式的根。它可能是零、一個或兩個實數根。沒有根代表光沒有擊中球體；一個根代表光與球體交於一點 (擦過 「grazing hit」)；兩個根代表光與球體交於兩點 (一點交於進入時、一點交於離開時)。在最後一個情況裡，我們想要兩個根之中較小的那個； *n* 與光離開觀測點的距離成正比，所以先擊中的會是較小的 *n* 。所以我們呼叫 ``minroot`` 。如果有一個根， ``sphere-intersect`` 回傳代表該點的 :math:`〈x_0+nx_r,y_0+ny_r,z_0+nz_r〉` 。

圖 9.5 的另外兩個函數， ``normal`` 與 ``sphere-normal`` 類比於 ``intersect`` 與 ``sphere-intersect`` 。要找到垂直於球體很簡單 –– 不過是從該點至球體中心的向量而已。

圖 9.6 示範了我們如何產生圖片； ``ray-test`` 定義了 38 個球體（不全都看的見）然後產生一張圖片，叫做 "sphere.pgm" 。

(譯註：PGM 可移植灰度圖格式 ，更多訊息參見 `wiki <http://en.wikipedia.org/wiki/Portable_graymap>`_ )

::

	(defun ray-test (&optional (res 1))
	  (setf *world* nil)
	  (defsphere 0 -300 -1200 200 .8)
	  (defsphere -80 -150 -1200 200 .7)
	  (defsphere 70 -100 -1200 200 .9)
	  (do ((x -2 (1+ x)))
	      ((> x 2))
	    (do ((z 2 (1+ z)))
	        ((> z 7))
	      (defsphere (* x 200) 300 (* z -400) 40 .75)))
	  (tracer (make-pathname :name "spheres.pgm") res))

**圖 9.6 使用光線追蹤器**

圖 9.7 是產生出來的圖片，其中 ``res`` 參數為 10。

.. figure:: https://github.com/acl-translation/acl-chinese/raw/master/images/Figure-9.7.png

**圖 9.7: 追蹤光線的圖**

一個實際的光線追蹤器可以產生更複雜的圖片，因為它會考慮更多，我們只考慮了單一光源至平面某一點。可能會有多個光源，每一個有不同的強度。它們通常不會在觀測點，在這個情況程式需要檢查至光源的向量是否與其他平面相交，這會在第一個相交的平面上產生陰影。將光源放置於觀測點讓我們不需要考慮這麼複雜的情況，因為我們看不見在陰影中的任何點。

一個實際的光線追蹤器不僅追蹤光第一個擊中的平面，也會加入其它平面的反射光。一個實際的光線追蹤器會是有顏色的，並可以模型化出透明或是閃耀的平面。但基本的算法會與圖 9.3 所展示的差不多，而許多改進只需要遞迴的使用同樣的成分。

一個實際的光線追蹤器可以是高度優化的。這裡給出的程式為了精簡寫成，甚至沒有如 Lisp 程式設計師會最佳化的那樣，就僅是一個光線追蹤器而已。僅加入型態與行內宣告 (13.3 節)就可以讓它變得兩倍以上快。

Chapter 9 總結 (Summary)
============================

1. Common Lisp 提供整數 (integers)、比值 (ratios)、浮點數 (floating-point numbers)以及複數 (complex numbers)。

2. 數字可以被約分或轉換 (converted)，而它們的位數 (components)可以被取出。

3. 用來比較數字的判斷式可以接受任意數量的參數，以及比較下一數對 (successive pairs) –– `/=` 函數除外，它是用來比較所有的數對 (pairs)。

4. Common Lisp 幾乎提供你在低階科學計算機可以看到的數值函數。同樣的函數普遍可應用在多種類型的數字上。

5. Fixnum 是小至可以塞入一個字 (word)的整數。它們在必要時會悄悄但花費昂貴地轉成大數 (bignum)。Common Lisp 提供最多四種浮點數。每一個浮點表示法的限制是實現相關的 (implementation-dependent)常數。

6. 一個光線追蹤器 (ray-tracer)通過追蹤光線來產生圖像，使得每一像素回到模擬的世界。

Chapter 9 練習 (Exercises)
==================================

1. 定義一個函數，接受一個實數列表，若且唯若 (iff)它們是非遞減 (nondecreasing)順序時回傳真。

2. 定義一個函數，接受一個整數 ``cents`` 並回傳四個值，將數字用 ``25-`` , ``10-`` , ``5-`` , ``1-`` 來顯示，使用最少數量的硬幣。(譯註: ``25-`` 是 25 美分，以此類推)

3. 一個遙遠的星球住著兩種生物， wigglies 與 wobblies 。 Wigglies 與 wobblies 唱歌一樣厲害。每年都有一個比賽來選出十大最佳歌手。下面是過去十年的結果:

+----------+---+---+---+---+---+---+---+---+---+----+
| YEAR     | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
+==========+===+===+===+===+===+===+===+===+===+====+
| WIGGLIES | 6 | 5 | 6 | 4 | 5 | 5 | 4 | 5 | 6 | 5  |
+----------+---+---+---+---+---+---+---+---+---+----+
| WOBBLIES | 4 | 5 | 4 | 6 | 5 | 5 | 6 | 5 | 4 | 5  |
+----------+---+---+---+---+---+---+---+---+---+----+

寫一個程式來模擬這樣的比賽。你的結果實際上有建議委員會每年選出 10 個最佳歌手嗎？

4. 定義一個函數，接受 8 個表示二維空間中兩個線段端點的實數，若線段沒有相交，則回傳假，或回傳兩個值表示相交點的 ``x`` 座標與 ``y`` 座標。

5. 假設 ``f`` 是一個接受一個 (實數) 參數的函數，而 ``min`` 與 ``max`` 是有著不同正負號的非零實數，使得 ``f`` 對於參數 ``i`` 有一個根 (回傳零)並滿足 ``min < i < max`` 。定義一個函數，接受四個參數， ``f`` , ``min`` , ``max`` 以及 ``epsilon`` ，並回傳一個 ``i`` 的近似值，準確至正負 ``epsilon`` 之內。

6. *Honer's method* 是一個有效率求出多項式的技巧。要找到 :math:`ax^3+bx^2+cx+d` 你對 ``x(x(ax+b)+c)+d`` 求值。定義一個函數，接受一個或多個參數 –– x 的值伴隨著 *n* 個實數，用來表示 ``(n-1)`` 次方的多項式的係數 –– 並用 *Honer's method* 計算出多項式的值。

譯註: `Honer's method on wiki <http://en.wikipedia.org/wiki/Horner's_method>`_

7. 你的 Common Lisp 實現使用了幾個位元來表示定數 (fixnum)？

8. 你的 Common Lisp 實現提供幾種不同的浮點數？

.. rubric:: 腳註

.. [1] 當 ``format`` 取整顯示時，它不保證會取成偶數或奇數。見 125 頁 (譯註: 7.4 節)。
