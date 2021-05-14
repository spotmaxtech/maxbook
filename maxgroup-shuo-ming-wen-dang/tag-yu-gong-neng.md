# Tag与功能

max\_group功能依赖于识别Autoscaling（AWS）或者伸缩组（阿里云）所配置的tag来实现不同的功能，以下为tag key-value及功能解释

<table>
  <thead>
    <tr>
      <th style="text-align:center">tag-key</th>
      <th style="text-align:center">tag-value</th>
      <th style="text-align:center">&#x529F;&#x80FD;</th>
      <th style="text-align:center">&#x7248;&#x672C;&#x652F;&#x6301;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">spotmax:detaching_delay_seconds</td>
      <td style="text-align:center">30</td>
      <td style="text-align:center">&#x5F53;&#x89E6;&#x53D1;spot&#x56DE;&#x6536;&#x65F6;&#xFF0C;&#x95F4;&#x9694;&#x591A;&#x5C11;&#x79D2;&#x540E;&#xFF0C;&#x5C06;&#x88AB;&#x56DE;&#x6536;&#x673A;&#x5668;&#x4ECE;asg&#x4E2D;detach&#xFF0C;&#x9ED8;&#x8BA4;&#x4E3A;30&#x79D2;</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:is_enable_preaction</td>
      <td style="text-align:center">true</td>
      <td style="text-align:center">&#x589E;&#x52A0;&#x6B64;tag&#x4E3A;&#x5F00;&#x542F;&#x96C6;&#x7FA4;&#x9632;&#x9000;&#x5316;&#x529F;&#x80FD;&#xFF0C;&#x6B64;&#x529F;&#x80FD;&#x4E3A;&#x9884;&#x6D4B;&#x5373;&#x5C06;&#x88AB;&#x56DE;&#x6536;&#x7684;&#x673A;&#x5668;&#xFF0C;&#x5E76;&#x63D0;&#x524D;&#x8FDB;&#x884C;&#x66F4;&#x66FF;&#x673A;&#x578B;&#x64CD;&#x4F5C;&#xFF0C;tag-value&#x4E3A;true&#x8868;&#x793A;&#x4E3A;&#x5F00;&#x542F;&#x6B64;&#x529F;&#x80FD;</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:max_num_of_terminated_one_time</td>
      <td style="text-align:center">2</td>
      <td style="text-align:center">&#x96C6;&#x7FA4;&#x9632;&#x9000;&#x5316;&#x529F;&#x80FD;&#x4E00;&#x6B21;&#x5173;&#x95ED;&#x7684;&#x6700;&#x5927;&#x673A;&#x5668;&#x6570;&#xFF0C;&#x66FF;&#x6362;&#x673A;&#x5668;&#x6267;&#x884C;&#x5206;&#x6279;&#x66FF;&#x6362;&#xFF0C;&#x6BCF;&#x6B21;&#x66FF;&#x6362;&#x7684;&#x6700;&#x5927;&#x6570;&#x91CF;</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:preaction_termination_delay_seconds</td>
      <td style="text-align:center">600</td>
      <td style="text-align:center">&#x96C6;&#x7FA4;&#x9632;&#x9000;&#x5316;&#x529F;&#x80FD;&#x6267;&#x884C;terminate&#x95F4;&#x9694;&#x65F6;&#x95F4;</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:preaction_detach_delay_seconds</td>
      <td style="text-align:center">600</td>
      <td style="text-align:center">&#x96C6;&#x7FA4;&#x9632;&#x9000;&#x5316;&#x529F;&#x80FD;&#x4E2D;&#xFF0C;&#x5C06;&#x88AB;&#x66FF;&#x6362;&#x673A;&#x5668;&#x95F4;&#x9694;&#x591A;&#x5C11;&#x79D2;&#x540E;&#xFF0C;&#x4F1A;&#x88AB;detach&#x51FA;asg</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:is_enable_od_fallback</td>
      <td style="text-align:center">true</td>
      <td style="text-align:center">&#x6B64;tag-value&#x4E3A;true&#x8868;&#x793A;&#xFF0C;&#x5728;&#x524D;&#x8FF0;&#x4E2D;&#x65AD;&#x9884;&#x8865;&#x507F;&#x673A;&#x5236;&#x4E2D;&#xFF0C;&#x5F53;&#x7ADE;&#x4EF7;&#x5B9E;&#x4F8B;&#x65E0;&#x6CD5;&#x83B7;&#x53D6;&#x65F6;&#xFF0C;&#x4F1A;&#x7528;&#x6309;&#x9700;&#x5B9E;&#x4F8B;&#x8865;&#x5145;</td>
      <td
      style="text-align:center">Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:persistence_dev</td>
      <td style="text-align:center">/dev/sdf</td>
      <td style="text-align:center">&#x6DFB;&#x52A0;&#x6B64;tag&#x53EF;&#x4EE5;&#x8FDB;&#x884C;ebs&#x7684;&#x6F02;&#x79FB;&#xFF0C;&#x65E0;&#x9ED8;&#x8BA4;&#x503C;&#xFF0C;tag-value&#x4E3A;&#x975E;root&#x76D8;&#x5728;instance&#x4E0A;&#x7684;&#x6620;&#x5C04;&#x8DEF;&#x5F84;&#xFF0C;&#x6682;&#x65F6;&#x4EC5;aws&#x5E73;&#x53F0;&#x652F;&#x6301;</td>
      <td
      style="text-align:center">Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:consul_port</td>
      <td style="text-align:center">8500</td>
      <td style="text-align:center">&#x914D;&#x7F6E;&#x6B64;&#x53C2;&#x6570;&#x4E3A;consul&#x652F;&#x6301;&#xFF0C;&#x65E0;&#x9ED8;&#x8BA4;&#x503C;&#xFF0C;tag-value&#x4E3A;consul
        agent&#x672C;&#x5730;&#x7AEF;&#x53E3;&#x53F7; &#x5728;&#x5B9E;&#x4F8B;&#x4E2D;&#x65AD;&#x5E76;&#x7ECF;&#x8FC7;detaching_delay_seconds&#x65F6;&#x95F4;&#x540E;&#xFF0C;&#x8BE5;&#x5B9E;&#x4F8B;&#x5C06;&#x4F1A;&#x4ECE;consul&#x7684;&#x670D;&#x52A1;&#x53D1;&#x73B0;&#x5217;&#x8868;&#x4E2D;&#x79FB;&#x9664;</td>
      <td
      style="text-align:center">Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:k8s_config_file_path</td>
      <td style="text-align:center">xxx/config</td>
      <td style="text-align:center">kubernetes &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x7528;&#x4E8E;&#x628A;&#x6743;&#x9650;&#x8D4B;&#x7ED9;max
        group</td>
      <td style="text-align:center">Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:k8s_node_drain_grace_second</td>
      <td style="text-align:center">600</td>
      <td style="text-align:center">node&#x4E0B;&#x7684;pod&#x79FB;&#x51FA;&#x5EF6;&#x8FDF;&#x65F6;&#x95F4;</td>
      <td
      style="text-align:center">Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:spot_price_limit</td>
      <td style="text-align:center">0.01 - 0.99</td>
      <td style="text-align:center">spot&#x4EF7;&#x683C;&#x9650;&#x5236;&#xFF0C;&#x4F8B;&#x5982; 0.9&#xFF0C;
        &#x5F53;spot&#x673A;&#x578B;&#x4EF7;&#x683C;&#x8D85;&#x8FC7;&#x6309;&#x9700;&#x673A;&#x578B;&#x4EF7;&#x683C;&#x7684;90%&#xFF0C;&#x4ECE;&#x66FF;&#x6362;&#x673A;&#x578B;&#x5217;&#x8868;&#x4E2D;&#x79FB;&#x51FA;&#x8FD9;&#x4E2A;&#x673A;&#x578B;</td>
      <td
      style="text-align:center">ali Lite/Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:fixed_desired_capacity</td>
      <td style="text-align:center">3</td>
      <td style="text-align:center">&#x91CD;&#x65B0;&#x8BBE;&#x7F6E;&#x4F38;&#x7F29;&#x7EC4;&#x7684;&#x673A;&#x5668;&#x7684;&#x671F;&#x671B;&#x503C;</td>
      <td
      style="text-align:center">Pro</td>
    </tr>
    <tr>
      <td style="text-align:center">spotmax:alt(num)</td>
      <td style="text-align:center">ecs.mn4.large</td>
      <td style="text-align:center">
        <p>&#x7528;&#x4E8E;&#x5F53;&#x4F38;&#x7F29;&#x7EC4;&#x5185;&#x7684;&#x673A;&#x5668;&#x6CA1;&#x6709;&#x65F6;&#xFF0C;&#x7528;&#x989D;&#x5916;&#x7684;&#x673A;&#x5668;&#x6765;&#x66FF;&#x6362;&#xFF0C;&#x4F8B;&#x5B50;&#xFF1A;</p>
        <p>(<b>key</b>:spotmax:alt<b>1</b>
        </p>
        <p><b>value</b>:ecs.mn4.large)</p>
        <p>(<b>key</b>:spotmax:alt<b>2</b>
        </p>
        <p><b>value</b>:ecs.n2.medium)</p>
      </td>
      <td style="text-align:center">Pro</td>
    </tr>
  </tbody>
</table>

**注：当启用 spotmax:k8s\_node\_drain\_option 时，建议将spotmax:detaching\_delay\_seconds 的tag-value设置为80-90之间，这样可以在保证新node ready情况下，将pod转移过去。**

