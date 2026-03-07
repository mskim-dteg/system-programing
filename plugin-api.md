# API/Plugin
## [GStreamer](https://gstreamer.freedesktop.org/)
### GObject를 기반으로 하는 object oriented C library로 multimedia data를 처리하기 위한 framework
  + member function, inheritance를 c로 구현
  + 수동 reference counting 방식의 object의 메모리 관리
    - g_object_ref(...) : reference count 증가
    - g_object_unref() : reference count 감소, 0이 되면 object의 메모리 해제
### [GstPipeline](https://gstreamer.freedesktop.org/documentation/gstreamer/gstpipeline.html?gi-language=c) : 여러 element(plugin)를 연결하여 multimedia data flow를 구성하는 단위

  ![gstreamer pipeline](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a5/GStreamer_example_pipeline.svg/960px-GStreamer_example_pipeline.svg.png)
  + 한 process에 여러개의 pipeline 가능
### [GstPlugin](https://gstreamer.freedesktop.org/documentation/gstreamer/gstplugin.html?gi-language=c#GstPlugin)
  + plugin은 shared library로 구현
  + 보통 자동으로 load됨
  + element factory를 제공하여 element의 생성과 초기화를 담당
### [GstElement](https://gstreamer.freedesktop.org/documentation/gstreamer/gstelement.html?gi-language=c) : multimedia data를 처리하는 모듈

  ![gstreamer element](https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/GStreamer_Technical_Overview.svg/500px-GStreamer_Technical_Overview.svg.png)
  + element는 property, signal, pad를 가질 수 있음
    - property : element의 동작을 제어하는 parameter
    - signal : element에서 발생하는 event
    - pad : element의 input/output interface
  + 예 : GObject → GstObject → GstElement → GstVideoEncoder → GstV4l2VideoEnc → v4l2h264enc
### [GstBuffer](https://gstreamer.freedesktop.org/documentation/gstreamer/gstbuffer.html?gi-language=c)
  + multimedia buffer의 abstract representation
  + element에서 생성, 처리된 후 pad를 통해 전달
  + [GstMemory](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmemory.html?gi-language=c)와 [GstMeta](https://gstreamer.freedesktop.org/documentation/gstreamer/gstmeta.html?gi-language=c)로 구성
    - GstMemory : multimedia data가 저장된 memory block에 대한자정보
    - GstMeta : multimedia data에 대한 추가 정보(metadata) : timestamp, object detection 결과, ...
### 장/단점
  + 장점
    - 필요한 기능을 plugin 형태로 쉽게 확장 가능
    - 다양한 기존 plugin 재사용 가능
  + 단점
    - 복잡한 구조
    - heavy dependency


## [Pipewire Simple Plugin API](https://docs.pipewire.org/page_spa.html)
### [Pipewire introduction](https://bootlin.com/blog/an-introduction-to-pipewire/)
  ![pipewire architecture](https://bootlin.com/wp-content/uploads/2022/06/schema-proxies.jpg)
### Design
  + Types
    - String types
    - Integer types
  + Error Codes : negative integer
### Plugins
  + Design Overview
    - No implicit dependencies, SPA is shipped as a set of header files that have no dependencies except for the standard C library.
    - Extensible; new API can be added with minimal effort, existing API can be updated and versioned
  + Plugin usage
    - Load the shared library.
    - Enumerate the available factories.
    - Enumerate the interfaces in each factory.
    - Instantiate the desired interface.
    - Use the interface-specific functions.
  + Open A Plugin
    ```c
    #define SPA_PLUGIN_DIR  /usr/lib64/spa-0.2/"
    void *hnd = dlopen(SPA_PLUGIN_DIR"/support/libspa-support.so", RTLD_NOW);
    ```
    ```c
    #define 	SPA_HANDLE_FACTORY_ENUM_FUNC_NAME   "spa_handle_factory_enum"
    typedef int(* spa_handle_factory_enum_func_t) (const struct spa_handle_factory **factory, uint32_t *index)
    spa_handle_factory_enum_func_t enum_func;
    enum_func = dlsym(hnd, SPA_HANDLE_FACTORY_ENUM_FUNC_NAME));
    ```
  + Enumerating Factories
    ```c
    struct spa_handle_factory {
      uint32_t version;
      const char *name;
      // ....
    };

    uint32_t i;
    const struct spa_handle_factory *factory = NULL;
    for (i = 0;;) {
      if (enum_func(&factory, &i) <= 0)
        break;
      // check name and version, introspect interfaces,
      // do something with the factory.
    }
    ```
  + Making A Handle
    - get the size of the required memory
      ```c
      struct spa_dict *extra_params = NULL;
      size_t size = spa_handle_factory_get_size(factory, extra_params);
      ```
    - allocate the memory and initialize the object in it
      ```c
      struct spa_handle {
        uint32_t version;
        int (*get_interface) (struct spa_handle *handle, const char *type, void **iface);
        int (*clear) (struct spa_handle *handle);
      };

      struct spa_handle *handle = (struct spa_handle *) calloc(1, size);
      spa_handle_factory_init(factory, handle, extra_params/*info*/,  NULL/*support*/, 0 /*n_support*/);
      ```
      support/n_support : handle이 의존하는 다른 handle의 interface를 제공하기 위한 parameter, 보통은 NULL/0으로 설정
  + Clearing An Object
    - clear handle
      ```c
      spa_handle_clear(handle);
      free(handle);
      ```
    - close shared library
      ```c
      dlclose(hnd);
      ```
  + Retrieving An Interface (예: log)
    ```c
    void *iface;
    spa_handle_get_interface(handle, SPA_NAME_SUPPORT_LOG, &iface);
    struct spa_log *log = iface;
    spa_log_warn(log, "Hello World!\n");
    ```
    ```c
    struct spa_log {
      struct spa_interface iface;
      enum spa_log_level level;
    };
    struct spa_interface {
      const char *type;
      uint32_t version;
      struct spa_callbacks cb;
    };
    struct spa_callbacks {
      const void *funcs;
      void *data;
    };
    ```
  + call interface method
    ```c
    struct spa_log_methods {
      uint32_t version;
      void (*log) (void *object, enum spa_log_level level, const char *file, int line, const char *func, const char *fmt, ...); // v0
      void (*logv) (void *object, enum spa_log_level level, const char *file, int line, const char *func, const char *fmt, va_list args); // v0
      void (*logt) (void *object, enum spa_log_level level, const struct spa_log_topic *topic, const char *file, int line, const char *func, const char *fmt, ...); // v1
      void (*logtv) (void *object, enum spa_log_level level, const struct spa_log_topic *topic, const char *file, int line, const char *func, const char *fmt, va_list args); // v1
      void (*topic_init) (void *object, struct spa_log_topic *topic); // v1
    };
    #define spa_callbacks_call(callbacks,type,method,vers,...)			\
    ({										\
      const type *_f = (const type *) (callbacks)->funcs;			\
      bool _res = SPA_CALLBACK_CHECK(_f,method,vers);				\
      if (SPA_LIKELY(_res))							\
        (_f->method)((callbacks)->data, ## __VA_ARGS__);		\
      _res;									\
    })
    ```
### POD(Plain Old Data)
* LTV(Length Type Value) format : Each POD is made of a 32 bits size followed by a 32 bits type field, followed by the POD contents
  ```c
  struct spa_pod {
    uint32_t size;          /* size of the body */
    uint32_t type;          /* a basic id of enum spa_type */
  };
  struct spa_pod_bool {
    struct spa_pod pod;
    int32_t value;
    int32_t _padding;
  };
  ```
  **The POD start is always aligned to 8 bytes : marhalling/unmarshalling이 필요없음**
* basic SPA types
  + SPA_TYPE_None: No value or a NULL pointer.
  + SPA_TYPE_Bool: A boolean value.
  + SPA_TYPE_Id: An enumerated value.
  + SPA_TYPE_Int, SPA_TYPE_Long, SPA_TYPE_Float, SPA_TYPE_Double: various numeral types, 32 and 64 bits.
  + SPA_TYPE_String: A string.
  + SPA_TYPE_Bytes: A byte array.
  + SPA_TYPE_Rectangle: A rectangle with width and height.
  + SPA_TYPE_Fraction: A fraction with numerator and denominator.
  + SPA_TYPE_Bitmap: An array of bits.
* container types
  + SPA_TYPE_Array: An array of equal sized objects.
  + SPA_TYPE_Struct: A collection of types and objects.
  + SPA_TYPE_Object: An object with properties.
  + SPA_TYPE_Sequence: A timed sequence of POD's.
* extra types
  + SPA_TYPE_Pointer: A typed pointer in memory.
  + SPA_TYPE_Fd: A file descriptor.
  + SPA_TYPE_Choice: A choice of values.
  + SPA_TYPE_Pod: A generic type for the POD itself
