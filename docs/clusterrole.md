# ClusterRole ve ClusterRoleBinding

ClusterRole ve ClusterRoleBinding, Kubernetes cluster'ında izinleri yönetmek için kullanılan öğelerdir. ClusterRole, bir kullanıcının ya da bir grubun Kubernetes cluster'ında yapabileceği işlemleri belirtir. Örneğin, bir ClusterRole ile bir kullanıcının pod'ları oluşturma, silebilme veya güncelleme gibi işlemleri yapabileceği belirtilebilir.

ClusterRoleBinding ise, bir ClusterRole ile bir kullanıcı veya bir grubu eşleştirir. Örneğin, bir ClusterRoleBinding ile bir kullanıcının pod'ları oluşturma, silme ve güncelleme gibi işlemleri yapabileceği belirtilebilir.

ClusterRole ve ClusterRoleBinding, Kubernetes cluster'ında izinleri yönetmek için kullanılırken, bu izinler namespace bazlı değil, cluster bazlı verilir. Bu nedenle, bir ClusterRole veya ClusterRoleBinding ile verilen izinler, tüm namespace'lerde geçerlidir.
