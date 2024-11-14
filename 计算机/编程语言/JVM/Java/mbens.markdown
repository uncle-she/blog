# MBeans

## tutorial

<https://docs.oracle.com/javase/tutorial/jmx/mbeans/standard.html>

## folder example

``` java
... = new ObjectName("foo.com:00=folder,01=subFolder,name=SomeBean");
```

<https://stackoverflow.com/questions/20669928/is-it-possible-to-create-jmx-subdomains>

## dynamic mbeans

``` java
import javax.management.Attribute;
import javax.management.AttributeList;
import javax.management.AttributeNotFoundException;
import javax.management.DynamicMBean;
import javax.management.MBeanAttributeInfo;
import javax.management.MBeanInfo;
import javax.management.MalformedObjectNameException;
import javax.management.ObjectName;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.Map;
import java.util.function.Supplier;

public class MetricsInfoDynamicMBean implements DynamicMBean {
    public static final Logger LOGGER = LoggerFactory.getLogger(MetricsInfoDynamicMBean.class);
    private final ObjectName objectName;
    private final Map<String, Supplier<Object>> metrics = new HashMap<>();
    private final Map<String, String> metricsDesc = new HashMap<>();

    public MetricsInfoDynamicMBean(String mbeanName) throws MalformedObjectNameException {
        this.objectName = new ObjectName(mbeanName);
    }

    public void setAttribute(String name, String desc, Supplier<Object> supplier) {
        this.metrics.put(name, supplier);
        this.metricsDesc.put(name, desc);
    }

    public ObjectName getName() {
        return this.objectName;
    }

    @Override
    public Object getAttribute(String attribute) throws AttributeNotFoundException {
        if (this.metrics.containsKey(attribute))
            return this.metrics.get(attribute).get();
        else
            throw new AttributeNotFoundException("Could not find attribute " + attribute);
    }

    @Override
    public AttributeList getAttributes(String[] attributes) {
        AttributeList list = new AttributeList();
        for (String name : attributes) {
            try {
                list.add(new Attribute(name, getAttribute(name)));
            } catch (Exception e) {
                LOGGER.warn("Error getting JMX attribute '{}'", name, e);
            }
        }
        return list;
    }

    @Override
    public MBeanInfo getMBeanInfo() {
        MBeanAttributeInfo[] attrs = new MBeanAttributeInfo[metrics.size()];
        int i = 0;
        for (Map.Entry<String, Supplier<Object>> entry : this.metrics.entrySet()) {
            String attribute = entry.getKey();
            Supplier<Object> value = entry.getValue();
            attrs[i] = new MBeanAttributeInfo(attribute,
                    value.get().getClass().getName(),
                    metricsDesc.getOrDefault(attribute, ""),
                    true,
                    false,
                    false);
            i += 1;
        }
        return new MBeanInfo(this.getClass().getName(), "", attrs, null, null, null);
    }

    @Override
    public Object invoke(String name, Object[] params, String[] sig) {
        throw new UnsupportedOperationException("Invoke not allowed.");
    }

    @Override
    public void setAttribute(Attribute attribute) {
        throw new UnsupportedOperationException("Set not allowed.");
    }

    @Override
    public AttributeList setAttributes(AttributeList list) {
        throw new UnsupportedOperationException("Set not allowed.");
    }


}
```

## 修改历史

- `2024.01.02` 创建文章
