# ConfigMap Objesi

ConfigMap objesi, Kubernetes cluster'ında yapılandırma bilgilerini saklamak için kullanılan bir nesnedir. ConfigMap objesi, yapılandırma bilgilerini key-value şeklinde saklar ve bu bilgileri bir pod veya bir serviste kullanılmak üzere açıklamaya (mount) yönelik olarak pod veya servise sağlar.

ConfigMap objesi, yapılandırma bilgilerinin saklandığı bir dosya olarak da oluşturulabilir. Bu dosya, ConfigMap objesi oluşturulurken bir manifest dosyası olarak belirtilebilir veya kubectl komutu ile de oluşturulabilir. Oluşturulan ConfigMap objesi, Kubernetes cluster'ında bir namespace altında saklanır ve daha sonra pod veya servislere açıklanarak kullanılabilir.

ConfigMap objesi, yapılandırma bilgilerinin saklandığı bir dosya olarak da oluşturulabilir. Bu dosya, ConfigMap objesi oluşturulurken bir manifest dosyası olarak belirtilebilir veya kubectl komutu ile de oluşturulabilir. Oluşturulan ConfigMap objesi, Kubernetes cluster'ında bir namespace altında saklanır ve daha sonra pod veya servislere açıklanarak kullanılabilir.
